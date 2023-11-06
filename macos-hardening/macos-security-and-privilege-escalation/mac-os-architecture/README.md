# macOS कर्नल और सिस्टम एक्सटेंशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## XNU कर्नल

**macOS** का मूल हिस्सा XNU है, जिसका अर्थ है "X is Not Unix". यह कर्नल मूल रूप से **Mach माइक्रोकर्नल** (जिसके बारे में बाद में चर्चा होगी), **और** Berkeley Software Distribution (**BSD**) के तत्वों से मिलकर बना है। XNU भी **I/O Kit** नामक एक सिस्टम के माध्यम से **कर्नल ड्राइवर्स के लिए एक प्लेटफ़ॉर्म प्रदान करता है**। XNU कर्नल Darwin ओपन सोर्स प्रोजेक्ट का हिस्सा है, जिसका मतलब है कि **इसका स्रोत कोड मुक्त रूप से उपलब्ध है**।

एक सुरक्षा शोधकर्ता या यूनिक्स डेवलपर के दृष्टिकोण से, **macOS** एक **FreeBSD** सिस्टम के बहुत ही **समान** लग सकता है जिसमें एक सुंदर GUI और कई कस्टम एप्लिकेशन्स होती हैं। BSD के लिए विकसित अधिकांश एप्लिकेशन्स को macOS परिवर्तन किए बिना कंपाइल और चलाया जा सकता है, क्योंकि macOS में यूनिक्स उपयोगकर्ताओं के लिए परिचित कमांड-लाइन उपकरण मौजूद होते हैं। हालांकि, XNU कर्नल में Mach शामिल होने के कारण, एक पारंपरिक यूनिक्स-जैसे सिस्टम और macOS के बीच कुछ महत्वपूर्ण अंतर होते हैं, और ये अंतर संभावित मुद्दों का कारण बन सकते हैं या अद्वितीय लाभ प्रदान कर सकते हैं।

XNU का ओपन सोर्स संस्करण: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach एक **माइक्रोकर्नल** है जो **UNIX-संगत** बनाने के लिए डिज़ाइन किया गया है। इसकी मुख्य डिज़ाइन सिद्धांतों में से एक था कि **कर्नल स्थान में चल रहे कोड की मात्रा को कम किया जाए** और इसके बजाय कई प्रकार के कर्नल कार्यों, जैसे फ़ाइल सिस्टम, नेटवर्किंग और I/O, को **यूज़र-स्तर के टास्क के रूप में चलाने की अनुमति दी जाए**।

XNU में, Mach **प्रोसेसर समय-अनुसूची, मल्टीटास्किंग और वर्चुअल मेमोरी प्रबंधन** जैसे कर्नल के आम लो-लेवल ऑपरेशनों के लिए जिम्मेदार है।

### BSD

XNU **कर्नल** में भी **FreeBSD** प्रोजेक्ट से प्राप्त कोड की एक महत्वपूर्ण मात्रा शामिल होती है। यह कोड Mach के साथ कर्नल का हिस्सा होता है, एक ही पता स्थान में। हालांकि, XNU में FreeBSD कोड मूल FreeBSD कोड से बहुत अलग हो सकता है क्योंकि Mach के साथ इसके संगतता को सुनिश्चित करने के लिए संशोधन किए जाने की आवश्यकता थी। FreeBSD निम्नलिखित कर्नल ऑपरेशन में योगदान देता है:

* प
#### IMG4

IMG4 फ़ाइल प्रारूप एक कंटेनर फ़ाइल प्रारूप है जिसे Apple अपने iOS और macOS उपकरणों में सुरक्षित रूप से फर्मवेयर संघटकों (जैसे कर्नेलकैश) को संग्रहीत और सत्यापित करने के लिए उपयोग करता है। IMG4 प्रारूप में एक हैडर और कई टैग होते हैं जो वास्तविक पेलोड (जैसे कर्नेल या बूटलोडर), एक हस्ताक्षर और एक सेट के मानिफेस्ट गुणों को संगठित करते हैं। यह प्रारूप ऊर्जात्मक सत्यापन का समर्थन करता है, जिससे उपकरण को फर्मवेयर संघटक की प्रामाणिकता और अखंडता की पुष्टि करने की अनुमति मिलती है इसे क्रियान्वित करने से पहले।

इसमें आमतौर पर निम्नलिखित घटक होते हैं:

* **पेलोड (IM4P)**:
* अक्सर संपीड़ित (LZFSE4, LZSS, ...)
* वैकल्पिक रूप से एन्क्रिप्टेड
* **मानिफेस्ट (IM4M)**:
* हस्ताक्षर समेत
* अतिरिक्त कुंजी / मान शब्दकोश
* **पुनर्स्थापन जानकारी (IM4R)**:
* यहां तक कि APNonce के रूप में भी जाना जाता है
* कुछ अपडेट को फिर से चलाने से रोकता है
* वैकल्पिक: आमतौर पर यह नहीं मिलता है

कर्नेलकैश को डीकंप्रेस करें:
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### कर्नलकैश संकेत

कभी-कभी Apple **कर्नलकैश** के साथ **संकेत** जारी करता है। आप [https://theapplewiki.com](https://theapplewiki.com/) पर दिए गए लिंक का पालन करके कुछ फर्मवेयर को संकेतों के साथ डाउनलोड कर सकते हैं।

### IPSW

ये Apple के **फर्मवेयर** हैं जिन्हें आप [**https://ipsw.me/**](https://ipsw.me/) से डाउनलोड कर सकते हैं। अन्य फ़ाइलों के बीच में इसमें **कर्नलकैश** शामिल होगा।\
फ़ाइलों को **अनज़िप** करके आप उन्हें निकाल सकते हैं।

फर्मवेयर को निकालने के बाद आपको एक फ़ाइल मिलेगी जैसे: **`kernelcache.release.iphone14`**। यह **IMG4** प्रारूप में है, आप इसके साथ दिलचस्प जानकारी को निकाल सकते हैं:

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
आप निम्नलिखित कमांड का उपयोग करके संकेतों के लिए निकाली गई kernelcache की जांच कर सकते हैं: **`nm -a kernelcache.release.iphone14.e | wc -l`**

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
## macOS कर्नल एक्सटेंशन्स

macOS कर्नल एक्सटेंशन्स (.kext) लोड करने के लिए **बहुत प्रतिबंधकारी** है क्योंकि इसके द्वारा चलाए जाने वाले कोड के उच्च अधिकार होते हैं। वास्तव में, डिफ़ॉल्ट रूप से यह लगभग असंभव है (जब तक कोई बाईपास नहीं मिल जाता है)।

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### macOS सिस्टम एक्सटेंशन्स

macOS ने कर्नल एक्सटेंशन्स का उपयोग करने की बजाय सिस्टम एक्सटेंशन्स बनाए हैं, जो कर्नल के साथ इंटरैक्ट करने के लिए उपयोगकर्ता स्तरीय API प्रदान करता है। इस तरीके से, डेवलपर्स कर्नल एक्सटेंशन्स का उपयोग करने से बच सकते हैं।

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## संदर्भ

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की सुविधा प्राप्त करना चाहते हैं? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को फ़ॉलो करके।**

</details>
