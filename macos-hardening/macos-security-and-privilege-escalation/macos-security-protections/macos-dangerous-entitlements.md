# macOS खतरनाक अधिकार (Entitlements) और TCC अनुमतियाँ

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram समूह**](https://t.me/peass) या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.**

</details>

{% hint style="warning" %}
ध्यान दें कि **`com.apple`** से शुरू होने वाले अधिकार (entitlements) तीसरे पक्ष को उपलब्ध नहीं हैं, केवल Apple उन्हें अनुदान दे सकता है।
{% endhint %}

## उच्च

### `com.apple.rootless.install.heritable`

अधिकार **`com.apple.rootless.install.heritable`** SIP को **बायपास** करने की अनुमति देता है। [इसे अधिक जानकारी के लिए देखें](macos-sip.md#com.apple.rootless.install.heritable).

### **`com.apple.rootless.install`**

अधिकार **`com.apple.rootless.install`** SIP को **बायपास** करने की अनुमति देता है। [इसे अधिक जानकारी के लिए देखें](macos-sip.md#com.apple.rootless.install).

### **`com.apple.system-task-ports` (पहले `task_for_pid-allow` कहा जाता था)**

यह अधिकार किसी भी प्रक्रिया के लिए **task port प्राप्त करने** की अनुमति देता है, कर्नेल को छोड़कर। [**इसे अधिक जानकारी के लिए देखें**](../mac-os-architecture/macos-ipc-inter-process-communication/).

### `com.apple.security.get-task-allow`

यह अधिकार अन्य प्रक्रियाओं को जिनके पास **`com.apple.security.cs.debugger`** अधिकार है, उस प्रक्रिया के task port को प्राप्त करने और **उस पर कोड इंजेक्ट करने** की अनुमति देता है जिसे इस अधिकार के साथ बाइनरी द्वारा चलाया जाता है। [**इसे अधिक जानकारी के लिए देखें**](../mac-os-architecture/macos-ipc-inter-process-communication/).

### `com.apple.security.cs.debugger`

डिबगिंग टूल अधिकार के साथ ऐप्स `task_for_pid()` को कॉल कर सकते हैं ताकि वे अनसाइन्ड और तीसरे पक्ष के ऐप्स के लिए एक वैध task port प्राप्त कर सकें जिनके पास `Get Task Allow` अधिकार `true` पर सेट है। हालांकि, डिबगिंग टूल अधिकार के साथ भी, एक डिबगर **task ports प्राप्त नहीं कर सकता** उन प्रक्रियाओं के जिनके पास `Get Task Allow` अधिकार **नहीं है**, और जो इसलिए System Integrity Protection द्वारा संरक्षित हैं। [**इसे अधिक जानकारी के लिए देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger).

### `com.apple.security.cs.disable-library-validation`

यह अधिकार **Apple द्वारा साइन किए गए या मुख्य निष्पादन योग्य के समान Team ID के साथ साइन किए गए बिना फ्रेमवर्क, प्लग-इन्स, या लाइब्रेरीज को लोड करने** की अनुमति देता है, इसलिए एक हमलावर किसी मनमानी लाइब्रेरी लोड का दुरुपयोग करके कोड इंजेक्ट कर सकता है। [**इसे अधिक जानकारी के लिए देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation).

### `com.apple.private.security.clear-library-validation`

यह अधिकार **`com.apple.security.cs.disable-library-validation`** के समान है लेकिन **इसके बजाय** लाइब्रेरी वैलिडेशन को **सीधे अक्षम करने** के बजाय, यह प्रक्रिया को **`csops` सिस्टम कॉल करने की अनुमति देता है ताकि वह इसे अक्षम कर सके**।\
[**इसे अधिक जानकारी के लिए देखें**](https://theevilbit.github.io/posts/com.apple.private.security.clear-library-validation/).

### `com.apple.security.cs.allow-dyld-environment-variables`

यह अधिकार **DYLD पर्यावरण चर (environment variables) का उपयोग करने** की अनुमति देता है जिसका उपयोग लाइब्रेरीज और कोड इंजेक्ट करने के लिए किया जा सकता है। [**इसे अधिक जानकारी के लिए देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables).

### `com.apple.private.tcc.manager` या `com.apple.rootless.storage`.`TCC`

[**इस ब्लॉग के अनुसार**](https://objective-see.org/blog/blog\_0x4C.html) **और** [**इस ब्लॉग के अनुसार**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/), ये अधिकार **TCC** डेटाबेस को **संशोधित करने** की अनुमति देते हैं।

### **`system.install.apple-software`** और **`system.install.apple-software.standar-user`**

ये अधिकार **बिना अनुमति मांगे सॉफ्टवेयर इंस्टॉल करने** की अनुमति देते हैं, जो **विशेषाधिकार वृद्धि (privilege escalation)** के लिए सहायक हो सकता है।

### `com.apple.private.security.kext-management`

अधिकार जो **कर्नेल से कर्नेल एक्सटेंशन लोड करने के लिए कहने** की आवश्यकता होती है।

### **`com.apple.private.icloud-account-access`**

अधिकार **`com.apple.private.icloud-account-access`** के साथ संभव है **`com.apple.iCloudHelper`** XPC सेवा के साथ संवाद करना जो **iCloud टोकन प्रदान करेगी**।

**iMovie** और **Garageband** में यह अधिकार था।

उस अधिकार से **iCloud टोकन प्राप्त करने के लिए एक्सप्लॉइट के बारे में अधिक जानकारी** के लिए टॉक देखें: [**#OBTS v5.0: "What Happens on your Mac, Stays on Apple's iCloud?!" - Wojciech Regula**](https://www.youtube.com/watch?v=\_6e2LhmxVc0)

### `com.apple.private.tcc.manager.check-by-audit-token`

TODO: मुझे नहीं पता कि यह क्या करने की अनुमति देता है

### `com.apple.private.apfs.revert-to-snapshot`

TODO: [**इस रिपोर्ट में**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **उल्लेख किया गया है कि इसका उपयोग SSV-संरक्षित सामग्री को रिबूट के बाद अपडेट करने के लिए किया जा सकता है। अगर आपको पता है तो कृपया PR भेजें!**

### `com.apple.private.apfs.create-sealed-snapshot`

TODO: [**इस रिपोर्ट में**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **उल्लेख किया गया है कि इसका उपयोग SSV-संरक्षित सामग्री को रिबूट के बाद अपडेट करने के लिए किया जा सकता है। अगर आपको पता है तो कृपया PR भेजें!**

### `keychain-access-groups`

यह अधिकार सूचीबद्ध करता है **keychain** समूह जिन तक एप्लिकेशन की पहुँच है:
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

**पूर्ण डिस्क एक्सेस** अनुमतियाँ प्रदान करता है, जो TCC की सबसे उच्च अनुमतियों में से एक है।

### **`kTCCServiceAppleEvents`**

ऐप को अन्य एप्लिकेशन्स को इवेंट्स भेजने की अनुमति देता है जो आमतौर पर **कार्यों को स्वचालित करने** के लिए उपयोग किए जाते हैं। अन्य एप्स को नियंत्रित करते हुए, यह उन एप्लिकेशन्स को दी गई अनुमतियों का दुरुपयोग कर सकता है।

जैसे कि उन्हें उपयोगकर्ता से उसका पासवर्ड मांगने के लिए बनाना:

{% code overflow="wrap" %}
```bash
osascript -e 'tell app "App Store" to activate' -e 'tell app "App Store" to activate' -e 'tell app "App Store" to display dialog "App Store requires your password to continue." & return & return default answer "" with icon 1 with hidden answer with title "App Store Alert"'
```
{% endcode %}

या उन्हें **मनमानी क्रियाएं** करने के लिए मजबूर करना।

### **`kTCCServiceEndpointSecurityClient`**

अन्य अनुमतियों के बीच, यह **उपयोगकर्ता के TCC डेटाबेस को लिखने** की अनुमति देता है।

### **`kTCCServiceSystemPolicySysAdminFiles`**

यह **`NFSHomeDirectory`** विशेषता को **बदलने** की अनुमति देता है, जो एक उपयोगकर्ता के होम फोल्डर पथ को बदलता है और इसलिए TCC को **बायपास** करने की अनुमति देता है।

### **`kTCCServiceSystemPolicyAppBundles`**

ऐप्स बंडल के अंदर (ऐप.app के अंदर) फाइलों को संशोधित करने की अनुमति देता है, जो **डिफ़ॉल्ट रूप से अनुमति नहीं है**।

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

_सिस्टम सेटिंग्स_ > _प्राइवेसी & सिक्योरिटी_ > _ऐप मैनेजमेंट_ में जाकर इस एक्सेस को कौन रखता है यह जांचना संभव है।

### `kTCCServiceAccessibility`

प्रक्रिया macOS की एक्सेसिबिलिटी सुविधाओं का **दुरुपयोग कर सकेगी**, जिसका मतलब है कि उदाहरण के लिए वह कीस्ट्रोक्स दबा सकेगा। इसलिए वह Finder जैसे ऐप को नियंत्रित करने के लिए एक्सेस का अनुरोध कर सकता है और इस अनुमति के साथ डायलॉग को मंजूरी दे सकता है।

## मध्यम

### `com.apple.security.cs.allow-jit`

यह अनुमति `mmap()` सिस्टम फंक्शन को `MAP_JIT` फ्लैग पास करके **लिखने योग्य और निष्पादन योग्य मेमोरी बनाने** की अनुमति देती है। [**इसके लिए अधिक जानकारी**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit) देखें।

### `com.apple.security.cs.allow-unsigned-executable-memory`

यह अनुमति **C कोड को ओवरराइड या पैच करने**, लंबे समय से प्रचलित **`NSCreateObjectFileImageFromMemory`** (जो मूल रूप से असुरक्षित है) का उपयोग करने, या **DVDPlayback** फ्रेमवर्क का उपयोग करने की अनुमति देती है। [**इसके लिए अधिक जानकारी**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory) देखें।

{% hint style="danger" %}
इस अनुमति को शामिल करने से आपके ऐप को मेमोरी-असुरक्षित कोड भाषाओं में सामान्य कमजोरियों का खतरा होता है। ध्यान से विचार करें कि क्या आपके ऐप को यह अपवाद चाहिए।
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

यह अनुमति अपनी खुद की निष्पादन योग्य फाइलों के खंडों को **संशोधित करने** की अनुमति देती है ताकि जबरन बाहर निकल सकें। [**इसके लिए अधिक जानकारी**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection) देखें।

{% hint style="danger" %}
निष्पादन योग्य मेमोरी सुरक्षा अनुमति को निष्क्रिय करना एक अत्यंत अनुमति है जो आपके ऐप से एक मौलिक सुरक्षा संरक्षण को हटाता है, जिससे एक हमलावर के लिए आपके ऐप के निष्पादन योग्य कोड को बिना पता लगाए पुनर्लेखन करना संभव हो जाता है। यदि संभव हो तो संकीर्ण अनुमतियों को प्राथमिकता दें।
{% endhint %}

### `com.apple.security.cs.allow-relative-library-loads`

TODO

### `com.apple.private.nullfs_allow`

यह अनुमति एक nullfs फाइल सिस्टम को माउंट करने की अनुमति देती है (जो डिफ़ॉल्ट रूप से निषिद्ध है)। टूल: [**mount\_nullfs**](https://github.com/JamaicanMoose/mount\_nullfs/tree/master).

### `kTCCServiceAll`

इस ब्लॉगपोस्ट के अनुसार, यह TCC अनुमति आमतौर पर इस रूप में पाई जाती है:
```
[Key] com.apple.private.tcc.allow-prompting
[Value]
[Array]
[String] kTCCServiceAll
```
प्रक्रिया को **सभी TCC अनुमतियों के लिए पूछने की अनुमति दें**।

### **`kTCCServicePostEvent`**

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, hacktricks repo** में PRs सबमिट करके और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में।

</details>
