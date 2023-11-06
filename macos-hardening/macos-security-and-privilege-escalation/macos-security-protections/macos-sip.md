# macOS SIP

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## **मूलभूत जानकारी**

**सिस्टम अखंडता सुरक्षा (SIP)** एक सुरक्षा प्रौद्योगिकी है जो macOS में कुछ सिस्टम निर्देशिकाओं को अनधिकृत पहुंच से सुरक्षित रखती है, यहां तक कि रूट उपयोगकर्ता के लिए भी। इससे इन निर्देशिकाओं में संशोधन, निर्माण या फ़ाइलों के हटाने जैसे परिवर्तनों की रोक होती है। SIP द्वारा संरक्षित मुख्य निर्देशिकाएं हैं:

* **/System**
* **/bin**
* **/sbin**
* **/usr**

इन निर्देशिकाओं और उनके उपनिर्देशिकाओं के लिए संरक्षण नियम **`/System/Library/Sandbox/rootless.conf`** फ़ाइल में निर्दिष्ट होते हैं। इस फ़ाइल में, तारक में आरंभ होने वाले पथ SIP की प्रतिबंधों के लिए अपवादों को प्रतिष्ठानित करते हैं।

उदाहरण के लिए, निम्नलिखित कॉन्फ़िगरेशन:
```javascript
/usr
* /usr/libexec/cups
* /usr/local
* /usr/share/man
```
इसका अर्थ है कि **`/usr`** निर्दिष्ट रूप से SIP द्वारा सुरक्षित होता है। हालांकि, तीन उपनिर्दिष्ट उपनिर्देशित उपनिर्देशितों (`/usr/libexec/cups`, `/usr/local`, और `/usr/share/man`) में संशोधनों की अनुमति है, क्योंकि उन्हें एक प्रमुख एस्ट्रिक (\*) के साथ सूचीबद्ध किया गया है।

SIP द्वारा किसी निर्देशिका या फ़ाइल की सुरक्षा की जांच करने के लिए, आप **`ls -lOd`** कमांड का उपयोग कर सकते हैं ताकि **`restricted`** या **`sunlnk`** ध्वज की मौजूदगी की जांच कर सकें। उदाहरण के लिए:
```bash
ls -lOd /usr/libexec/cups
drwxr-xr-x  11 root  wheel  sunlnk 352 May 13 00:29 /usr/libexec/cups
```
इस मामले में, **`sunlnk`** फ्लैग यह दर्शाता है कि `/usr/libexec/cups` निर्देशिका स्वयं **हटाई नहीं जा सकती**, हालांकि इसके भीतर के फ़ाइलें बनाई, संशोधित या हटाई जा सकती हैं।

दूसरे हाथ:
```bash
ls -lOd /usr/libexec
drwxr-xr-x  338 root  wheel  restricted 10816 May 13 00:29 /usr/libexec
```
यहां, **`restricted`** ध्वज यह दर्शाता है कि `/usr/libexec` निर्देशिका SIP द्वारा सुरक्षित है। SIP से सुरक्षित निर्देशिका में, फ़ाइलें नहीं बनाई जा सकती हैं, संशोधित की जा सकती हैं या हटाई जा सकती हैं।

### SIP स्थिति

आप निम्नलिखित कमांड के साथ जांच सकते हैं कि SIP आपके सिस्टम पर सक्षम है या नहीं:
```bash
csrutil status
```
यदि आप SIP को अक्षम करना चाहते हैं, तो आपको अपने कंप्यूटर को रिकवरी मोड में पुनः चालू करना होगा (स्टार्टअप के दौरान Command+R दबाकर), फिर निम्नलिखित कमांड को निष्पादित करें:
```bash
csrutil disable
```
यदि आप SIP सक्षम रखना चाहते हैं लेकिन डीबगिंग सुरक्षा को हटाना चाहते हैं, तो आप निम्नलिखित के साथ ऐसा कर सकते हैं:
```bash
csrutil enable --without debug
```
### अन्य प्रतिबंध

SIP द्वारा कई अन्य प्रतिबंध भी लगाए जाते हैं। उदाहरण के लिए, इसे **साइन नहीं किए गए कर्नल एक्सटेंशन्स (kexts) को लोड करने** और macOS सिस्टम प्रोसेस की **डीबगिंग** से रोका जाता है। इसके अलावा, यह टूल जैसे dtrace को भी सिस्टम प्रोसेस की जांच करने से रोकता है।

## SIP बाईपास

### मूल्य

यदि कोई हमलावर SIP को बाईपास करने में सफल होता है, तो वह निम्नलिखित कमाई करेगा:

* सभी उपयोगकर्ताओं के मेल, संदेश, Safari इतिहास... को पढ़ें
* वेबकैम, माइक्रोफ़ोन या किसी भी चीज़ के लिए अनुमतियाँ प्रदान करें (SIP से सुरक्षित TCC डेटाबेस पर सीधे लिखकर)
* स्थायित्व: वह SIP से सुरक्षित स्थान पर मैलवेयर सहेज सकता है और कोई भी टूल इसे हटा नहीं सकेगा। इसके अलावा, वह MRT के साथ छेड़छाड़ कर सकता है।
* कर्नल एक्सटेंशन्स लोड करने की सरलता (इसके लिए अन्य कठिन सुरक्षा सुरक्षा भी हैं)।

### इंस्टॉलर पैकेज

**Apple के प्रमाणित कर्ता द्वारा साइन किए गए इंस्टॉलर पैकेज** इसकी सुरक्षा को छोड़ सकते हैं। इसका मतलब है कि यदि साधारण डेवलपर्स द्वारा साइन किए गए पैकेज SIP संरक्षित निर्देशिकाओं को संशोधित करने का प्रयास करें, तो वे ब्लॉक हो जाएंगे।

### अस्तित्वहीन SIP फ़ाइल

एक संभावित छेद है कि यदि कोई फ़ाइल **`rootless.conf` में निर्दिष्ट की गई होती है लेकिन वर्तमान में मौजूद नहीं है**, तो उसे बनाया जा सकता है। मैलवेयर इसे सिस्टम पर स्थायित्व स्थापित करने के लिए उपयोग कर सकता है। उदाहरण के लिए, यदि `rootless.conf` में सूचीबद्ध है लेकिन मौजूद नहीं है, तो एक दुष्ट कार्यक्रम `/System/Library/LaunchDaemons` में एक .plist फ़ाइल बना सकता है।

### com.apple.rootless.install.heritable

{% hint style="danger" %}
इंटाइटलमेंट **`com.apple.rootless.install.heritable`** SIP को बाईपास करने की अनुमति देता है
{% endhint %}

[**इस ब्लॉग पोस्ट के अनुसार**](https://www.microsoft.com/en-us/security/blog/2021/10/28/microsoft-finds-new-macos-vulnerability-shrootless-that-could-bypass-system-integrity-protection/), मैकओएस की सिस्टम इंटेग्रिटी प्रोटेक्शन (SIP) मेकेनिज़्म में एक सुरक्षा कमजोरी, 'श्रूटलेस' सुरक्षा कमजोरी के नाम से जानी जाती है। यह सुरक्षा कमजोरी **`system_installd`** डेमन के आसपास होती है, जिसमें एक इंटाइटलमेंट, **`com.apple.rootless.install.heritable`**, होती है, जो इसके किसी भी चाइल्ड प्रोसेस को SIP की फ़ाइल सिस्टम प्रतिबंधों को बाईपास करने की अनुमति देती है।

**`system_installd`** डेमन Apple द्वारा साइन किए गए पैकेज (.pkg फ़ाइल) की स्थापना के दौरान **पोस्ट-स्थापना** स्क्रिप्ट को चलाता है। इन स्क्रिप्ट्स को डिफ़ॉल्ट शेल **`zsh`** द्वारा निष्पादित किया जाता है, जो अपने पासवर्ड नहीं मांगता है, यदि यह मौजूद होता है, यहां तक कि गैर-सक्रिय मोड में भी। हमलावर इस व्यवहार का शोषण कर सकते हैं: एक दुष्ट `/etc/zshenv` फ़ाइल बनाकर और **`system_installd`** को `zsh` को आमंत्रित करने के लिए प्रतीक्षा करके, वे उपकरण पर विभिन्न कार्रवाई कर सकते हैं।

इसके अलावा, पाया गया है कि **`
|   |   स्नैपशॉट:                  FAA23E0C-791C-43FF-B0E7-0E1C0810AC61
|   |   स्नैपशॉट डिस्क:             disk3s1s1
<strong>|   |   स्नैपशॉट माउंट पॉइंट:      /
</strong><strong>|   |   स्नैपशॉट सील्ड:           हाँ
</strong>[...]
+-> वॉल्यूम डिस्क3s5 281959B7-07A1-4940-BDDF-6419360F3327
|   ---------------------------------------------------
|   APFS वॉल्यूम डिस्क (भूमिका):   डिस्क3s5 (डेटा)
|   नाम:                      Macintosh HD - डेटा (केस-असंवेदी)
<strong>    |   माउंट पॉइंट:               /System/Volumes/Data
</strong><strong>    |   उपयोग की क्षमता:         412071784448 B (412.1 जीबी)
</strong>    |   सील्ड:                    नहीं
|   फ़ाइलवॉल्ट:                 हाँ (अनलॉक किया गया)
</code></pre>

पिछले आउटपुट में दिखाया गया है कि **उपयोगकर्ता-पहुँच योग्य स्थान** `/System/Volumes/Data` के तहत माउंट होते हैं।

इसके अलावा, **macOS सिस्टम वॉल्यूम स्नैपशॉट** `/` में माउंट होता है और यह **सील्ड** होता है (ओएस द्वारा ऊपरी रूप से एन्क्रिप्ट किया जाता है)। इसलिए, यदि SIP को बाइपास किया जाता है और इसे संशोधित किया जाता है, तो **ओएस और बूट नहीं होगा**।

यह भी संभव है कि सील सक्षम है यह सत्यापित करने के लिए निम्नलिखित को चलाने से:
```bash
csrutil authenticated-root status
Authenticated Root status: enabled
```
इसके अलावा, स्नैपशॉट डिस्क भी **केवल पढ़ने योग्य** रूप में माउंट किया जाता है:
```
mount
/dev/disk3s1s1 on / (apfs, sealed, local, read-only, journaled)
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें,** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके।**

</details>
