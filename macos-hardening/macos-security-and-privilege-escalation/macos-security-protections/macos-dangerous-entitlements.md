# macOS खतरनाक अधिकार

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

{% hint style="warning" %}
ध्यान दें कि **`com.apple`** से शुरू होने वाले अधिकार तृतीय-पक्षों के लिए उपलब्ध नहीं होते हैं, केवल Apple उन्हें प्रदान कर सकता है।
{% endhint %}

## उच्च

### `com.apple.rootless.install.heritable`

अधिकार **`com.apple.rootless.install.heritable`** को **SIP को दौर करने की अनुमति** देता है। अधिक जानकारी के लिए [यह देखें](macos-sip.md#com.apple.rootless.install.heritable)।

### **`com.apple.rootless.install`**

अधिकार **`com.apple.rootless.install`** को **SIP को दौर करने की अनुमति** देता है। अधिक जानकारी के लिए [यह देखें](macos-sip.md#com.apple.rootless.install)।

### **`com.apple.system-task-ports` (पहले `task_for_pid-allow` कहलाता था)**

यह अधिकार किसी भी प्रक्रिया के लिए **कार्य पोर्ट प्राप्त करने की अनुमति** देता है, केवल कर्नल को छोड़कर। अधिक जानकारी के लिए [**यह देखें**](../mac-os-architecture/macos-ipc-inter-process-communication/)।

### `com.apple.security.get-task-allow`

यह अधिकार **`com.apple.security.cs.debugger`** अधिकार वाले अन्य प्रक्रियाओं को इस अधिकार वाले बाइनरी द्वारा चलाई गई प्रक्रिया का कार्य पोर्ट प्राप्त करने और इस पर कोड संचालित करने की अनुमति देता है। अधिक जानकारी के लिए [**यह देखें**](../mac-os-architecture/macos-ipc-inter-process-communication/)।

### `com.apple.security.cs.debugger`

डीबगिंग टूल अधिकार वाले ऐप्स को `task_for_pid()` कॉल करके `Get Task Allow` अधिकार को `true` पर सेट करने वाले अनसाइन्ड और थर्ड-पार्टी ऐप्स के लिए एक मान्य कार्य पोर्ट प्राप्त करने की अनुमति देता है। हालांकि, डीबगिंग टूल अधिकार के साथ भी, डीबगर **उन प्रक्रियाओं के कार्य पोर्ट्स को प्राप्त नहीं कर सकता** है जिनके पास `Get Task Allow` अधिकार नहीं हैं, और जो इसलिए सिस्टम अखंडता संरक्षण द्वारा संरक्षित हैं। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger)।

### `com.apple.security.cs.disable-library-validation`

यह अधिकार **Apple द्वारा हस्ताक्षरित या मुख्य क्रियाप्रणाली द्वारा हस्ताक्षरित न होने वाले** फ्रेमवर्क, प्लगइन या लाइब्रेरी लोड करने की अनुमति देता है, इसलिए किसी भी अनियमित लाइब्रेरी लोड को उपयोगकर्ता कोड संचालित करने के लिए एक हमलावर्धी लाइब्रेरी लोड कर सकता है। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation)।

### `com.apple.private.security.clear-library-validation`

यह अधिकार **`com.apple.security.cs.disable-library-validation`** के बहुत ही समान है, लेकिन **सीधे अक्षम करने** के बजाय, इसे प्रक्रिया को इसे अक्षम करने के लिए `csops` सिस्टम कॉल करने की अनुमति देता है।\
अधिक जानकारी के लिए [**यह देखें**](https://theevilbit.github.io/posts/com.apple.private.security.clear
### `com.apple.private.apfs.create-sealed-snapshot`

कार्यान्वयन के बाद SSV संरक्षित सामग्री को अद्यतन करने के लिए इसका उपयोग किया जा सकता है, जैसा कि [**इस रिपोर्ट**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) में उल्लेख किया गया है। यदि आप जानते हैं कि इसे कैसे भेजें, तो कृपया एक पीआर भेजें!

### `keychain-access-groups`

इस अधिकारांत सूची में कुंजिका समूहों को संग्रहीत करता है जिनका एप्लिकेशन का पहुंच होता है:
```xml
<key>keychain-access-groups</key>
<array>
<string>ichat</string>
<string>apple</string>
<string>appleaccount</string>
<string>InternetAccounts</string>
<string>IMCore</string>
</array>
```
### **`kTCCServiceSystemPolicyAllFiles`**

यह **पूर्ण डिस्क एक्सेस** अनुमतियाँ प्रदान करता है, यह TCC की सबसे उच्च अनुमतियों में से एक है।

### **`kTCCServiceAppleEvents`**

इस अनुमति के द्वारा ऐप को अन्य ऐप्स को इवेंट भेजने की अनुमति होती है जो **कार्यों को स्वचालित करने** के लिए आमतौर पर उपयोग की जाती है। अन्य ऐप्स को नियुक्त करके, यह अनुमतियाँ दी गई हैं उनका दुरुपयोग कर सकता है।

### **`kTCCServiceSystemPolicySysAdminFiles`**

इस अनुमति के द्वारा एक उपयोगकर्ता के **`NFSHomeDirectory`** गुणक को बदलने की अनुमति होती है जो उसके होम फ़ोल्डर को बदलता है और इसलिए TCC को **दुर्गम करने** की अनुमति होती है।

### **`kTCCServiceSystemPolicyAppBundles`**

ऐप्स बंडल (ऐप.ऐप के भीतर) के भीतर फ़ाइलों को संशोधित करने की अनुमति देता है, जो **डिफ़ॉल्ट रूप से अनुमति नहीं है**।

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## मध्यम

### `com.apple.security.cs.allow-jit`

इस अनुमति के द्वारा `mmap()` सिस्टम फ़ंक्शन को `MAP_JIT` फ़्लैग पास करके **लिखने और निष्पादित करने योग्य मेमोरी** बनाई जा सकती है। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit)।

### `com.apple.security.cs.allow-unsigned-executable-memory`

इस अनुमति के द्वारा C कोड को **ओवरराइड या पैच करने** की अनुमति होती है, लंबे समय से विचारशीलता वाले **`NSCreateObjectFileImageFromMemory`** का उपयोग करने की अनुमति होती है (जो मूलतः असुरक्षित है), या **DVDPlayback** framework का उपयोग करने की अनुमति होती है। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory)।

{% hint style="danger" %}
इस अनुमति को शामिल करने से आपका ऐप मेमोरी-असुरक्षित कोड भाषाओं में सामान्य संकटों के लिए अनुरक्षित हो जाता है। सावधानीपूर्वक विचार करें कि क्या आपके ऐप को इस अपवाद की आवश्यकता है।
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

इस अनुमति के द्वारा ऐप अपनी खुद की एक्सेक्यूटेबल फ़ाइलों के खंडों को **डिस्क पर संशोधित** करने की अनुमति होती है ताकि वह बलपूर्वक बंद हो सके। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection)।

{% hint style="danger" %}
डिसेबल एक्सेक्यूटेबल मेमोरी संरक्षण अनुमति एक अत्यधिक अनुमति है जो आपके ऐप से एक मौलिक सुरक्षा संरक्षण को हटा देती है, जिससे हमलावर्धक आपके ऐप को बिना पता चले उसके एक्सेक्यूटेबल कोड को पुनर्लेखित करने की संभावना होती है। संभव होने पर संकीर्ण अनुमतियों का प्रयोग करें।
{% endhint %}

### `com.apple.security.cs.allow-relative-library-loads`

TODO

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
