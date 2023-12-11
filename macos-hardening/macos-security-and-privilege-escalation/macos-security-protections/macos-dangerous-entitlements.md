# macOS खतरनाक अधिकार

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें और** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके** अपना योगदान दें।

</details>

{% hint style="warning" %}
ध्यान दें कि **`com.apple`** से शुरू होने वाले अधिकार तृतीय-पक्षों के लिए उपलब्ध नहीं होते हैं, केवल Apple उन्हें प्रदान कर सकता है।
{% endhint %}

## उच्च

### `com.apple.rootless.install.heritable`

अधिकार **`com.apple.rootless.install.heritable`** को **SIP को दौर करने** की अनुमति देता है। अधिक जानकारी के लिए [यह देखें](macos-sip.md#com.apple.rootless.install.heritable)।

### **`com.apple.rootless.install`**

अधिकार **`com.apple.rootless.install`** को **SIP को दौर करने** की अनुमति देता है। अधिक जानकारी के लिए [यह देखें](macos-sip.md#com.apple.rootless.install)।

### **`com.apple.system-task-ports` (पहले `task_for_pid-allow` कहलाता था)**

यह अधिकार किसी भी प्रक्रिया के लिए **कार्य पोर्ट प्राप्त करने** की अनुमति देता है, केवल कर्नल को छोड़कर। अधिक जानकारी के लिए [**यह देखें**](../mac-os-architecture/macos-ipc-inter-process-communication/)।

### `com.apple.security.get-task-allow`

यह अधिकार अन्य प्रक्रियाओं को **`com.apple.security.cs.debugger`** अधिकार वाले उपकरण के साथ इस अधिकार वाले बाइनरी द्वारा चलाई जाने वाली प्रक्रिया के कार्य पोर्ट प्राप्त करने और इस पर कोड संचालित करने की अनुमति देता है। अधिक जानकारी के लिए [**यह देखें**](../mac-os-architecture/macos-ipc-inter-process-communication/)।

### `com.apple.security.cs.debugger`

डीबगिंग टूल अधिकार वाले ऐप्स को `task_for_pid()` कॉल करके `Get Task Allow` अधिकार को `true` पर सेट करने वाले अप्रमाणित और थर्ड-पार्टी ऐप्स के लिए मान्य कार्य पोर्ट प्राप्त करने की अनुमति देता है। हालांकि, डीबगिंग टूल अधिकार के साथ भी, डीबगर **केवल उन प्रक्रियाओं के कार्य पोर्ट प्राप्त नहीं कर सकता** है जिनके पास `Get Task Allow` अधिकार नहीं हैं, और जो कि इसलिए सिस्टम अखंडता संरक्षण द्वारा संरक्षित हैं। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger)।

### `com.apple.security.cs.disable-library-validation`

यह अधिकार **Apple द्वारा हस्ताक्षरित या मुख्य executable के साथ समान टीम आईडी से हस्ताक्षरित न होने वाले** फ्रेमवर्क, प्लगइन या लाइब्रेरी लोड करने की अनुमति देता है, इसलिए किसी भी अनियमित लाइब्रेरी लोड को उपयोग करके एक हमलावर्धी कोड संचालित कर सकता है। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation)।

### `com.apple.private.security.clear-library-validation`

यह अधिकार **`com.apple.security.cs.disable-library-validation`** के बहुत समान है, लेकिन **सीधे अक्षम करने** के बजाय प्रक्रिया को इसे अक्षम करने के लिए `csops` सिस्टम कॉल करने क
### `com.apple.private.tcc.manager.check-by-audit-token`

यह क्या करने की अनुमति देता है, मुझे नहीं पता है।

### `com.apple.private.apfs.revert-to-snapshot`

यहां [**इस रिपोर्ट**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **में उल्लेख किया गया है कि इसका उपयोग** बूट के बाद SSV संरक्षित सामग्री को अपडेट करने के लिए किया जा सकता है। यदि आप जानते हैं कि यह कैसे करें, तो कृपया एक पीआर भेजें!

### `com.apple.private.apfs.create-sealed-snapshot`

यहां [**इस रिपोर्ट**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **में उल्लेख किया गया है कि इसका उपयोग** बूट के बाद SSV संरक्षित सामग्री को अपडेट करने के लिए किया जा सकता है। यदि आप जानते हैं कि यह कैसे करें, तो कृपया एक पीआर भेजें!

### `keychain-access-groups`

इस अनुमति सूची में **keychain** समूहों को सूचीबद्ध किया गया है जिनका एप्लिकेशन को पहुँच है:
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

यह **पूर्ण डिस्क एक्सेस** अनुमतियाँ देता है, जो TCC की सबसे उच्च अनुमतियों में से एक है।

### **`kTCCServiceAppleEvents`**

इसे ऐप को दूसरे ऐप्स को इवेंट भेजने की अनुमति देता है जो **कार्यों को स्वचालित करने** के लिए आमतौर पर उपयोग किए जाते हैं। अन्य ऐप्स को नियंत्रण करके, यह उन अन्य ऐप्स को दी गई अनुमतियों का दुरुपयोग कर सकता है।

जैसे कि यह उनसे उपयोगकर्ता से पासवर्ड पूछने के लिए उन्हें बना सकता है:

{% code overflow="wrap" %}
```bash
osascript -e 'tell app "App Store" to activate' -e 'tell app "App Store" to activate' -e 'tell app "App Store" to display dialog "App Store requires your password to continue." & return & return default answer "" with icon 1 with hidden answer with title "App Store Alert"'
```
{% endcode %}

या उन्हें **विचित्र कार्रवाई** करने के लिए मजबूर करना।

### **`kTCCServiceEndpointSecurityClient`**

यह अन्य अनुमतियों के बीच, **उपयोगकर्ताओं के TCC डेटाबेस को लिखने** की अनुमति देता है।

### **`kTCCServiceSystemPolicySysAdminFiles`**

इसे बदलने की अनुमति देता है **`NFSHomeDirectory`** एट्रिब्यूट को एक उपयोगकर्ता का बदलने वाला उपयोगकर्ता जो अपने होम फ़ोल्डर पथ को बदलता है और इसलिए **TCC को दौर करने की अनुमति देता है**।

### **`kTCCServiceSystemPolicyAppBundles`**

ऐप्स बंडल (ऐप्प के अंदर) के अंदर फ़ाइलें संशोधित करने की अनुमति देता है, जो **डिफ़ॉल्ट रूप से अनुमति नहीं है**।

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

यह जांचना संभव है कि _सिस्टम सेटिंग्स_ > _गोपनीयता और सुरक्षा_ > _ऐप प्रबंधन_ में कौन सी अनुमति है।

## मध्यम

### `com.apple.security.cs.allow-jit`

इस अधिकारदाता को **लिखने और निष्पादन योग्य मेमोरी बनाने** की अनुमति होती है, `mmap()` सिस्टम फ़ंक्शन को `MAP_JIT` फ़्लैग पास करके। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit)।

### `com.apple.security.cs.allow-unsigned-executable-memory`

इस अधिकारदाता को **C कोड को ओवरराइड या पैच करने** की अनुमति होती है, लंबे समय से अप्रचलित **`NSCreateObjectFileImageFromMemory`** का उपयोग करें (जो मूलतः असुरक्षित है), या **DVDPlayback** फ़्रेमवर्क का उपयोग करें। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory)।

{% hint style="danger" %}
इस अधिकारदाता को शामिल करने से आपका ऐप मेमोरी-असुरक्षित कोड भाषाओं में सामान्य सुरक्षा खतरों के लिए अनुरक्षित हो जाता है। सावधानीपूर्वक विचार करें कि क्या आपके ऐप को इस अपवाद की आवश्यकता है।
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

इस अधिकारदाता को अपनी खुद की एक्सीक्यूटेबल फ़ाइल के खंडों को **डिस्क पर संशोधित** करने की अनुमति होती है, जिससे बलपूर्वक बाहर निकलने के लिए। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection)।

{% hint style="danger" %}
डिसेबल एक्सीक्यूटेबल मेमोरी संरक्षण अधिकारदाता एक अत्यधिक अधिकारदाता है जो आपके ऐप से एक मौलिक सुरक्षा संरक्षण को हटा देता है, जिससे हमलावर आपके ऐप को बिना पता चले उसके एक्सीक्यूटेबल कोड को पुनर्लेखित करने की संभावना होती है। संभव हो सके तो संकीर्ण अधिकारदाता का प्रयास करें।
{% endhint %}

### `com.apple.security.cs.allow-relative-library-loads`

TODO

### `com.apple.private.nullfs_allow`

इस अधिकारदाता को एक nullfs फ़ाइल सिस्टम (डिफ़ॉल्ट रूप से निषिद्ध) माउंट करने की अनुमति होती है। टूल: [**mount\_nullfs**](https://github.com/JamaicanMoose/mount\_nullfs/tree/master)।

### `kTCCServiceAll`

इस ब्लॉगपोस्ट के अनुसार, यह TCC अनुमति आमतौर पर इस रूप में पाई जाती है:
```
[Key] com.apple.private.tcc.allow-prompting
[Value]
[Array]
[String] kTCCServiceAll
```
**सभी TCC अनुमतियों के लिए प्रक्रिया से पूछने की अनुमति दें।**

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपनी जानकारी साझा करें।**

</details>
