# macOS गंदा NIB

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपनी ट्रिक्स साझा करें।**

</details>

**यह तकनीक पोस्ट से ली गई है** [**https://blog.xpnsec.com/dirtynib/**](https://blog.xpnsec.com/dirtynib/)

## मूलभूत जानकारी

NIB फ़ाइलें Apple के विकास पारिस्थितिकी में उपयोग होती हैं ताकि एक एप्लिकेशन में **उपयोगकर्ता इंटरफ़ेस (UI) तत्वों** को परिभाषित किया जा सके और उनके इंटरैक्शन को प्रस्तुत किया जा सके। Interface Builder उपकरण के साथ बनाए गए इनमें खिड़कियाँ, बटन और पाठ क्षेत्र जैसे **सिलियराइज़्ड ऑब्जेक्ट्स** शामिल होते हैं, जो डिज़ाइन किए गए UI को प्रस्तुत करने के लिए रनटाइम पर लोड होते हैं। हालांकि अभी भी उपयोग में हैं, Apple ने अधिक विज़ुअल प्रतिनिधित्व के लिए Storyboards की सिफारिश की है जो एक एप्लिकेशन के UI फ्लो को प्रदर्शित करने के लिए होता है।

{% hint style="danger" %}
इसके अलावा, **NIB फ़ाइलें** अन्यायपूर्ण आदेशों को चलाने के लिए भी उपयोग की जा सकती हैं और यदि NIB फ़ाइल को एक ऐप में संशोधित किया जाता है, तो **Gatekeeper अभी भी ऐप को निष्पादित करने की अनुमति देगा**, इसलिए वे ऐप्लिकेशन के भीतर अन्यायपूर्ण आदेशों को चलाने के लिए उपयोग किए जा सकते हैं।
{% endhint %}

## गंदा NIB इंजेक्शन <a href="#dirtynib" id="dirtynib"></a>

सबसे पहले हमें एक नई NIB फ़ाइल बनानी होगी, हम XCode का उपयोग इसके अधिकांश निर्माण के लिए करेंगे। हम इंटरफ़ेस में एक ऑब्जेक्ट जोड़कर शुरू करते हैं और उसे NSAppleScript के लिए कक्षा सेट करते हैं:

<figure><img src="../../../.gitbook/assets/image (681).png" alt="" width="380"><figcaption></figcaption></figure>

ऑब्जेक्ट के लिए हमें प्राथमिक `source` संपत्ति सेट करनी होगी, जिसे हम उपयोगकर्ता परिभाषित रनटाइम गुणों का उपयोग करके कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (682).png" alt="" width="563"><figcaption></figcaption></figure>

यह हमारे कोड निष्पादन गैजेट सेटअप करता है, जो बस एक अनुरोध पर **AppleScript चलाएगा**। AppleScript के निष्पादन को वास्तव में ट्रिगर करने के लिए, हम अभी केवल एक बटन जोड़ेंगे (आप इसके साथ आविष्कारशील हो सकते हैं ;). बटन हमें उन्हीं `Apple Script` ऑब्जेक्ट से बाइंड करेगा जिसे हमने अभी बनाया है, और **`executeAndReturnError:` सेलेक्टर को आह्वान करेगा**:

<figure><img src="../../../.gitbook/assets/image (683).png" alt="" width="563"><figcaption></figcaption></figure>

टेस्टिंग के लिए हम केवल Apple Script का उपयोग करेंगे:
```bash
set theDialogText to "PWND"
display dialog theDialogText
```
और यदि हम इसे XCode डीबगर में चलाते हैं और बटन पर क्लिक करते हैं:

<figure><img src="../../../.gitbook/assets/image (684).png" alt="" width="563"><figcaption></figcaption></figure>

हमारी AppleScript कोड को NIB से चलाने की क्षमता के साथ, हमें अगला लक्ष्य चाहिए। हमारे प्राथमिक डेमो के लिए चुनें Pages, जो कि एक Apple एप्लिकेशन है और निश्चित रूप से हमारे द्वारा संशोधित नहीं होना चाहिए।

हम पहले एप्लिकेशन की एक प्रतिलिपि `/tmp/` में ले लेंगे:
```bash
cp -a -X /Applications/Pages.app /tmp/
```
तब हम ऐप्लिकेशन को लॉन्च करेंगे ताकि कोई भी गेटकीपर समस्या न हो और चीजें कैश की जा सकें:
```bash
open -W -g -j /Applications/Pages.app
```
पहली बार ऐप को लॉन्च करने (और मारने) के बाद, हमें अपनी DirtyNIB फ़ाइल के साथ एक मौजूदा NIB फ़ाइल को अधिलेखित करने की आवश्यकता होगी। डेमो के उद्देश्य के लिए, हम सिर्फ About Panel NIB को अधिलेखित करेंगे ताकि हम नियंत्रण कर सकें निष्पादन:
```bash
cp /tmp/Dirty.nib /tmp/Pages.app/Contents/Resources/Base.lproj/TMAAboutPanel.nib
```
एक बार जब हमने nib को अधिलेखित कर दिया है, तो हम `About` मेनू आइटम का चयन करके निष्पादन को ट्रिगर कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (685).png" alt="" width="563"><figcaption></figcaption></figure>

अगर हम Pages को थोड़ा और ध्यान से देखें, तो हम देखेंगे कि इसमें उपयोगकर्ता के फ़ोटो तक पहुंच की अनुमति देने के लिए एक निजी entitlement है:

<figure><img src="../../../.gitbook/assets/image (686).png" alt="" width="479"><figcaption></figcaption></figure>

इसलिए हम अपने POC को परीक्षण के लिए डाल सकते हैं **अपने AppleScript को संशोधित करके उपयोगकर्ता से पूछे बिना फ़ोटो चुरा लेने** के लिए: 

{% code overflow="wrap" %}
```applescript
use framework "Cocoa"
use framework "Foundation"

set grabbed to current application's NSData's dataWithContentsOfFile:"/Users/xpn/Pictures/Photos Library.photoslibrary/originals/6/68CD9A98-E591-4D39-B038-E1B3F982C902.gif"

grabbed's writeToFile:"/Users/xpn/Library/Containers/com.apple.iWork.Pages/Data/wtf.gif" atomically:1
```
{% endcode %}

{% hint style="danger" %}
[**खतरनाक .xib फ़ाइल जो विभिन्न कोड को चला सकती है उदाहरण।**](https://gist.github.com/xpn/16bfbe5a3f64fedfcc1822d0562636b4)
{% endhint %}

## लॉन्च सीमाएं

ये मुख्य रूप से **अपने अपेक्षित स्थानों के बाहर एप्लिकेशनों को चलाने से रोकते हैं**, इसलिए अगर आप एक ऐप्लिकेशन की कॉपी को `/tmp` में कॉपी करते हैं तो आप उसे चला नहीं पाएंगे।\
[**इस पोस्ट में अधिक जानकारी पाएं**](../macos-security-protections/#launch-constraints)**.**

हालांकि, फ़ाइल **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`** को पार्स करके आप अभी भी **लॉन्च सीमाओं द्वारा संरक्षित नहीं होने वाले ऐप्लिकेशन** ढूंढ सकते हैं, इसलिए आप अभी भी **विभिन्न स्थानों में NIB फ़ाइलें इंजेक्ट कर सकते हैं** (इन ऐप्स को कैसे ढूंढें इसके लिए पिछले लिंक की जांच करें)।

## अतिरिक्त सुरक्षा

macOS Somona से, कुछ सुरक्षा उपलब्ध है जो **ऐप्स के अंदर लिखने से रोकती हैं**। हालांकि, यदि आप बाइनरी की कॉपी चलाने से पहले Contents फ़ोल्डर का नाम बदल दें, तो आप इस सुरक्षा को अनदेखा कर सकते हैं:

1. `CarPlay Simulator.app` की एक कॉपी को `/tmp/` में ले जाएं
2. `/tmp/Carplay Simulator.app/Contents` को `/tmp/CarPlay Simulator.app/NotCon` में नाम बदलें
3. `/tmp/CarPlay Simulator.app/NotCon/MacOS/CarPlay Simulator` बाइनरी को चलाएं ताकि Gatekeeper में कैश हो जाएं
4. `NotCon/Resources/Base.lproj/MainMenu.nib` को हमारी `Dirty.nib` फ़ाइल से अधिलेखित करें
5. `/tmp/CarPlay Simulator.app/Contents` में नाम बदलें
6. `CarPlay Simulator.app` को फिर से चलाएं

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी में काम करते हैं**? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की आवश्यकता है**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **[💬](https://emojipedia.org/speech-balloon/) Discord समूह** या **[telegram समूह](https://t.me/peass)** में शामिल हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में**

</details>
