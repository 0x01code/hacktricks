# macOS SIP

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, hacktricks repo** को PRs सबमिट करके [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## **मूल जानकारी**

**System Integrity Protection (SIP)** macOS में एक सुरक्षा प्रौद्योगिकी है जो कुछ सिस्टम डायरेक्टरीज को अनधिकृत पहुँच से बचाती है, यहाँ तक कि रूट यूजर के लिए भी। यह इन डायरेक्टरीज में परिवर्तनों को रोकती है, जिसमें फाइलों का निर्माण, परिवर्तन, या हटाना शामिल है। SIP द्वारा संरक्षित मुख्य डायरेक्टरीज हैं:

* **/System**
* **/bin**
* **/sbin**
* **/usr**

इन डायरेक्टरीज और उनकी सबडायरेक्टरीज के लिए सुरक्षा नियम **`/System/Library/Sandbox/rootless.conf`** फाइल में निर्दिष्ट हैं। इस फाइल में, एस्टेरिस्क (\*) के साथ शुरू होने वाले पथ SIP की प्रतिबंधों के अपवाद को दर्शाते हैं।

उदाहरण के लिए, निम्नलिखित कॉन्फ़िगरेशन:
```javascript
/usr
* /usr/libexec/cups
* /usr/local
* /usr/share/man
```
संकेत करता है कि **`/usr`** निर्देशिका आम तौर पर SIP द्वारा संरक्षित होती है। हालांकि, तीन उपनिर्देशिकाओं में संशोधन की अनुमति है (`/usr/libexec/cups`, `/usr/local`, और `/usr/share/man`), क्योंकि वे एक अग्रणी तारांकन (\*) के साथ सूचीबद्ध हैं।

यह जांचने के लिए कि कोई निर्देशिका या फ़ाइल SIP द्वारा संरक्षित है या नहीं, आप **`ls -lOd`** कमांड का उपयोग कर सकते हैं **`restricted`** या **`sunlnk`** फ्लैग की उपस्थिति की जांच के लिए। उदाहरण के लिए:
```bash
ls -lOd /usr/libexec/cups
drwxr-xr-x  11 root  wheel  sunlnk 352 May 13 00:29 /usr/libexec/cups
```
इस मामले में, **`sunlnk`** फ्लैग का तात्पर्य है कि `/usr/libexec/cups` डायरेक्टरी स्वयं **हटाई नहीं जा सकती**, हालांकि इसके अंदर की फाइलों को बनाया, संशोधित, या हटाया जा सकता है।

दूसरी ओर:
```bash
ls -lOd /usr/libexec
drwxr-xr-x  338 root  wheel  restricted 10816 May 13 00:29 /usr/libexec
```
यहाँ, **`restricted`** फ्लैग दर्शाता है कि `/usr/libexec` डायरेक्टरी SIP द्वारा सुरक्षित है। SIP-सुरक्षित डायरेक्टरी में, फाइलों को बनाया, संशोधित, या हटाया नहीं जा सकता है।

इसके अलावा, यदि किसी फाइल में **`com.apple.rootless`** विस्तारित **एट्रिब्यूट** होता है, तो वह फाइल भी **SIP द्वारा सुरक्षित** होगी।

**SIP अन्य रूट क्रियाओं को भी सीमित करता है** जैसे:

* अविश्वसनीय कर्नेल एक्सटेंशन्स लोड करना
* Apple-हस्ताक्षरित प्रक्रियाओं के लिए टास्क-पोर्ट्स प्राप्त करना
* NVRAM वेरिएबल्स में संशोधन करना
* कर्नेल डिबगिंग की अनुमति देना

विकल्प nvram वेरिएबल में एक बिटफ्लैग के रूप में रखे जाते हैं (`csr-active-config` Intel पर और `lp-sip0` ARM के लिए बूटेड डिवाइस ट्री से पढ़ा जाता है)। आप फ्लैग्स को XNU सोर्स कोड में `csr.sh` में देख सकते हैं:

<figure><img src="../../../.gitbook/assets/image (720).png" alt=""><figcaption></figcaption></figure>

### SIP स्थिति

आप निम्नलिखित कमांड के साथ जांच सकते हैं कि आपके सिस्टम पर SIP सक्षम है या नहीं:
```bash
csrutil status
```
यदि आपको SIP अक्षम करना है, तो आपको अपने कंप्यूटर को रिकवरी मोड में पुनः आरंभ करना होगा (स्टार्टअप के दौरान Command+R दबाकर), फिर निम्नलिखित कमांड को निष्पादित करें:
```bash
csrutil disable
```
यदि आप SIP सक्षम रखना चाहते हैं लेकिन डिबगिंग सुरक्षा को हटाना चाहते हैं, तो आप इसे निम्नलिखित के साथ कर सकते हैं:
```bash
csrutil enable --without debug
```
### अन्य प्रतिबंध

SIP कई अन्य प्रतिबंध भी लगाता है। उदाहरण के लिए, यह **unsigned kernel extensions** (kexts) को लोड करने की अनुमति नहीं देता और macOS सिस्टम प्रोसेसेस की **debugging** को रोकता है। यह dtrace जैसे टूल्स को सिस्टम प्रोसेसेस की जांच करने से भी रोकता है।

## SIP Bypasses

यदि कोई हमलावर SIP को बायपास करने में सफल होता है, तो वह यह कर सकता है:

* सभी उपयोगकर्ताओं के मेल, संदेश, Safari इतिहास... पढ़ें
* वेबकैम, माइक्रोफोन या कुछ भी के लिए अनुमतियाँ प्रदान करें (SIP संरक्षित TCC डेटाबेस पर सीधे लिखकर)
* Persistence: वह मैलवेयर को SIP संरक्षित स्थान पर सहेज सकता है और तब भी toot उसे हटा नहीं पाएगा। इसके अलावा वह MRT के साथ छेड़छाड़ कर सकता है।
* Kernel extensions को लोड करने में आसानी (इसके लिए अन्य hardcore सुरक्षा भी मौजूद हैं)।

### Installer Packages

**Apple के प्रमाणपत्र के साथ हस्ताक्षरित Installer packages** इसकी सुरक्षा को बायपास कर सकते हैं। इसका मतलब है कि मानक डेवलपर्स द्वारा हस्ताक्षरित पैकेज भी अवरुद्ध हो जाएंगे यदि वे SIP-संरक्षित निर्देशिकाओं को संशोधित करने का प्रयास करते हैं।

### Inexistent SIP file

एक संभावित छेद यह है कि यदि कोई फ़ाइल **`rootless.conf` में निर्दिष्ट है लेकिन वर्तमान में मौजूद नहीं है**, तो इसे बनाया जा सकता है। मैलवेयर इसका फायदा उठाकर सिस्टम पर **persistence स्थापित** कर सकता है। उदाहरण के लिए, एक दुर्भावनापूर्ण प्रोग्राम `/System/Library/LaunchDaemons` में एक .plist फ़ाइल बना सकता है यदि यह `rootless.conf` में सूचीबद्ध है लेकिन मौजूद नहीं है।

### com.apple.rootless.install.heritable

{% hint style="danger" %}
प्रमाणपत्र **`com.apple.rootless.install.heritable`** SIP को बायपास करने की अनुमति देता है
{% endhint %}

[**इस ब्लॉग पोस्ट से शोधकर्ताओं**](https://www.microsoft.com/en-us/security/blog/2021/10/28/microsoft-finds-new-macos-vulnerability-shrootless-that-could-bypass-system-integrity-protection/) ने macOS के System Integrity Protection (SIP) तंत्र में एक कमजोरी की खोज की, जिसे 'Shrootless' कमजोरी कहा जाता है। इस कमजोरी का केंद्र **`system_installd`** डेमॉन है, जिसमें एक प्रमाणपत्र, **`com.apple.rootless.install.heritable`**, होता है, जो इसके चाइल्ड प्रोसेसेस को SIP की फाइल सिस्टम प्रतिबंधों को बायपास करने की अनुमति देता है।

**`system_installd`** डेमॉन **Apple** द्वारा हस्ताक्षरित पैकेजों को स्थापित करेगा।

शोधकर्ताओं ने पाया कि Apple-हस्ताक्षरित पैकेज (.pkg फ़ाइल) की स्थापना के दौरान, **`system_installd`** पैकेज में शामिल किसी भी **post-install** स्क्रिप्ट को **चलाता है**। ये स्क्रिप्ट डिफ़ॉल्ट शेल, **`zsh`** द्वारा निष्पादित की जाती हैं, जो गैर-इंटरैक्टिव मोड में भी, यदि **`/etc/zshenv`** फ़ाइल मौजूद है, तो उससे स्वचालित रूप से **चलाता है**। इस व्यवहार का लाभ उठाकर हमलावर: एक दुर्भावनापूर्ण `/etc/zshenv` फ़ाइल बनाकर और **`system_installd` के `zsh` को आमंत्रित करने की प्रतीक्षा करके**, वे डिवाइस पर मनमाने ऑपरेशन कर सकते हैं।

इसके अलावा, यह पता चला कि **`/etc/zshenv` का उपयोग एक सामान्य हमला तकनीक के रूप में किया जा सकता है**, केवल SIP बायपास के लिए नहीं। प्रत्येक उपयोगकर्ता प्रोफ़ाइल में एक `~/.zshenv` फ़ाइल होती है, जो `/etc/zshenv` की तरह ही काम करती है लेकिन इसके लिए रूट अनुमतियाँ की आवश्यकता नहीं होती। यह फ़ाइल प्रत्येक बार `zsh` शुरू होने पर एक persistence तंत्र के रूप में उपयोग की जा सकती है, या एक विशेषाधिकार वृद्धि तंत्र के रूप में। यदि एक व्यवस्थापक उपयोगकर्ता `sudo -s` या `sudo <command>` का उपयोग करके रूट में उन्नत होता है, तो `~/.zshenv` फ़ाइल ट्रिगर हो जाएगी, प्रभावी रूप से रूट में उन्नत हो जाएगी।

[**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/) में यह पता चला कि वही **`system_installd`** प्रक्रिया अभी भी दुरुपयोग की जा सकती है क्योंकि यह **post-install स्क्रिप्ट को `/tmp` के अंदर SIP द्वारा संरक्षित एक यादृच्छिक नामित फ़ोल्डर के अंदर रख रही थी**। बात यह है कि **`/tmp` स्वयं SIP द्वारा संरक्षित नहीं है**, इसलिए यह संभव था कि **वर्चुअल इमेज को इस पर माउंट** किया जाए, फिर **installer** वहाँ **post-install स्क्रिप्ट** डालेगा, **वर्चुअल इमेज को अनमाउंट** करेगा, **सभी फ़ोल्डर्स को पुनः बनाएगा** और **post installation** स्क्रिप्ट को **payload** के साथ जोड़ देगा जिसे निष्पादित करना है।

### **com.apple.rootless.install**

{% hint style="danger" %}
प्रमाणपत्र **`com.apple.rootless.install`** SIP को बायपास करने की अनुमति देता है
{% endhint %}

[**CVE-2022-26712**](https://jhftss.github.io/CVE-2022-26712-The-POC-For-SIP-Bypass-Is-Even-Tweetable/) से सिस्टम XPC सेवा `/System/Library/PrivateFrameworks/ShoveService.framework/Versions/A/XPCServices/SystemShoveService.xpc` में प्रमाणपत्र **`com.apple.rootless.install`** होता है, जो प्रक्रिया को SIP प्रतिबंधों को बायपास करने की अनुमति देता है। यह फ़ाइलों को बिना किसी सुरक्षा जांच के स्थानांतरित करने की विधि को भी **प्रकट करता है**।

## Sealed System Snapshots

Sealed System Snapshots एक विशेषता है जिसे Apple ने **macOS Big Sur (macOS 11)** में अपने **System Integrity Protection (SIP)** तंत्र के एक हिस्से के रूप में पेश किया था, जो अतिरिक्त सुरक्षा और सिस्टम स्थिरता प्रदान करने के लिए है। ये मूल रूप से सिस्टम वॉल्यूम के read-only संस्करण हैं।

यहाँ एक विस्तृत नज़र है:

1. **Immutable System**: Sealed System Snapshots macOS सिस्टम वॉल्यूम को "immutable" बनाते हैं, अर्थात इसे संशोधित नहीं किया जा सकता। यह सिस्टम में किसी भी अनधिकृत या आकस्मिक परिवर्तनों को रोकता है जो सुरक्षा या सिस्टम स्थिरता को समझौता कर सकते हैं।
2. **System Software Updates**: जब आप macOS अपडेट्स या अपग्रेड्स स्थापित करते हैं, macOS एक नया सिस्टम स्नैपशॉट बनाता है। macOS स्टार्टअप वॉल्यूम फिर **APFS (Apple File System)** का उपयोग करके इस नए स्नैपशॉट पर स्विच करता है। अपडेट्स लागू करने की पूरी प्रक्रिया सुरक्षित और अधिक विश्वसनीय हो जाती है क्योंकि सिस्टम हमेशा अपडेट के दौरान कुछ गलत होने पर पिछले स्नैपशॉट पर वापस जा सकता है।
3. **Data Separation**: macOS Catalina में प
<pre><code>|   |   स्नैपशॉट:                  FAA23E0C-791C-43FF-B0E7-0E1C0810AC61
|   |   स्नैपशॉट डिस्क:             disk3s1s1
<strong>|   |   स्नैपशॉट माउंट पॉइंट:      /
</strong><strong>|   |   स्नैपशॉट सील्ड:           हाँ
</strong>[...]
+-> वॉल्यूम disk3s5 281959B7-07A1-4940-BDDF-6419360F3327
|   ---------------------------------------------------
|   APFS वॉल्यूम डिस्क (रोल):   disk3s5 (डेटा)
|   नाम:                      Macintosh HD - डेटा (केस-इनसेंसिटिव)
<strong>    |   माउंट पॉइंट:               /System/Volumes/Data
</strong><strong>    |   क्षमता का उपयोग:         412071784448 B (412.1 GB)
</strong>    |   सील्ड:                    नहीं
|   FileVault:                 हाँ (अनलॉक्ड)
</code></pre>

पिछले आउटपुट में यह देखा जा सकता है कि **उपयोगकर्ता-सुलभ स्थान** `/System/Volumes/Data` के अंतर्गत माउंट किए गए हैं।

इसके अलावा, **macOS सिस्टम वॉल्यूम स्नैपशॉट** `/` में माउंट किया गया है और यह **सील्ड** है (ओएस द्वारा क्रिप्टोग्राफिक रूप से हस्ताक्षरित)। इसलिए, यदि SIP को बायपास किया जाता है और इसे संशोधित किया जाता है, तो **ओएस अब और बूट नहीं होगा**।

यह भी संभव है कि **सील सक्षम है यह जांचने के लिए** निम्नलिखित कमांड चलाकर:
```bash
csrutil authenticated-root status
Authenticated Root status: enabled
```
इसके अलावा, स्नैपशॉट डिस्क को **केवल-पढ़ने के लिए** माउंट किया गया है:
```
mount
/dev/disk3s1s1 on / (apfs, sealed, local, read-only, journaled)
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का अनुसरण करें।**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके।**

</details>
