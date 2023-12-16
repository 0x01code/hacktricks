# macOS खतरनाक अधिकार

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

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

यह अधिकार **`com.apple.security.cs.debugger`** अधिकार वाले अन्य प्रक्रियाओं को इस अधिकार वाले बाइनरी द्वारा चलाई गई प्रक्रिया का कार्य पोर्ट प्राप्त करने और इस पर कोड संचोदित करने की अनुमति देता है। अधिक जानकारी के लिए [**यह देखें**](../mac-os-architecture/macos-ipc-inter-process-communication/)।

### `com.apple.security.cs.debugger`

डीबगिंग टूल अधिकार वाले ऐप्स को `task_for_pid()` कॉल करके `Get Task Allow` अधिकार को `true` पर सेट करने वाले अप्रमाणित और थर्ड-पार्टी ऐप्स के लिए मान्य कार्य पोर्ट प्राप्त करने की अनुमति देता है। हालांकि, डीबगिंग टूल अधिकार के साथ भी, डीबगर **उन प्रक्रियाओं के कार्य पोर्ट्स को प्राप्त नहीं कर सकता** है जिनके पास `Get Task Allow` अधिकार नहीं हैं, और जो इसलिए सिस्टम अखंडता संरक्षण द्वारा संरक्षित हैं। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger)।

### `com.apple.security.cs.disable-library-validation`

यह अधिकार **Apple द्वारा हस्ताक्षरित या मुख्य executable के साथ समान टीम आईडी से हस्ताक्षरित न होने वाले** फ्रेमवर्क, प्लगइन या लाइब्रेरी लोड करने की अनुमति देता है, इसलिए किसी भी अनियमित लाइब्रेरी लोड को उपयोग करके कोड संचोदित किया जा सकता है। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation)।

### `com.apple.private.security.clear-library-validation`

यह अधिकार **`com.apple.security.cs.disable-library-validation`** के बहुत समान है, लेकिन **सीधे अक्षम करने** के बजाय प्रक्रिया को इसे अक्षम करने के लिए `csops` सिस्टम कॉल करने की अनुमति देता ह
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

यह **पूर्ण डिस्क एक्सेस** अनुमतियाँ देता है, यह TCC की सबसे उच्च अनुमतियों में से एक है जो आपके पास हो सकती है।

### **`kTCCServiceAppleEvents`**

इसके द्वारा ऐप को अन्य ऐप्स को इवेंट भेजने की अनुमति होती है जो **कार्यों को स्वचालित करने** के लिए आमतौर पर उपयोग की जाती हैं। अन्य ऐप्स को नियंत्रण करके, यह उन अन्य ऐप्स को दी गई अनुमतियों का दुरुपयोग कर सकता है।

जैसे कि यह उनसे उपयोगकर्ता से उनका पासवर्ड पूछने के लिए कह सकता है:

{% code overflow="wrap" %}
```bash
osascript -e 'tell app "App Store" to activate' -e 'tell app "App Store" to activate' -e 'tell app "App Store" to display dialog "App Store requires your password to continue." & return & return default answer "" with icon 1 with hidden answer with title "App Store Alert"'
```
{% endcode %}

या उन्हें **विचित्र कार्रवाई** करने के लिए मजबूर करना।

### **`kTCCServiceEndpointSecurityClient`**

यह अन्य अनुमतियों के बीच, **उपयोगकर्ताओं के TCC डेटाबेस को लिखने** की अनुमति देता है।

### **`kTCCServiceSystemPolicySysAdminFiles`**

इसे अनुमति देता है कि उपयोगकर्ता के **होम फ़ोल्डर पथ को बदलने वाले उपयोगकर्ता के `NFSHomeDirectory`** विशेषता को **बदलें** और इसलिए **TCC को दौर करने की अनुमति देता है**।

### **`kTCCServiceSystemPolicyAppBundles`**

ऐप्स बंडल (ऐप्प के अंदर) के अंदर फ़ाइलें संशोधित करने की अनुमति देता है, जो **डिफ़ॉल्ट रूप से अनुमति नहीं है**।

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

यह जांचना संभव है कि _सिस्टम सेटिंग्स_ > _गोपनीयता और सुरक्षा_ > _ऐप प्रबंधन_ में कौन इस पहुंच के साथ है।

## मध्यम

### `com.apple.security.cs.allow-jit`

इस अनुमति की अनुमति देती है कि **लिखने और निष्पादन योग्य मेमोरी बनाई जा सके**, `mmap()` सिस्टम फ़ंक्शन को `MAP_JIT` फ़्लैग पास करके। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit)।

### `com.apple.security.cs.allow-unsigned-executable-memory`

इस अनुमति की अनुमति देती है कि **C कोड को ओवरराइड या पैच किया जा सके**, पुरानी तरीके से विस्तारित **`NSCreateObjectFileImageFromMemory`** का उपयोग किया जाए (जो मूलतः असुरक्षित है), या **DVDPlayback** फ़्रेमवर्क का उपयोग किया जाए। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory)।

{% hint style="danger" %}
इस अनुमति को शामिल करने से आपका ऐप मेमोरी-असुरक्षित कोड भाषाओं में सामान्य सुरक्षा खतरों के लिए अव exposed हो जाता है। ध्यान से विचार करें कि क्या आपके ऐप को इस अपवाद की आवश्यकता है।
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

इस अनुमति की अनुमति देती है कि ऐप अपनी खुद की निष्पादन योग्य फ़ाइलों के खंडों को बदल सके ताकि वह बलपूर्वक बंद हो सके। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection)।

{% hint style="danger" %}
डिसेबल निष्पादन योग्य मेमोरी संरक्षण अनुमति एक अत्यधिक अनुमति है जो आपके ऐप से एक मौलिक सुरक्षा संरक्षण को हटा देती है, जिससे हमलावर आपके ऐप को बिना पता चले उसके निष्पादन योग्य कोड को पुनर्लेखित करने की संभावना होती है। संभव होने पर संकीर्ण अनुमतियों का प्राथमिकता दें।
{% endhint %}

### `com.apple.security.cs.allow-relative-library-loads`

TODO

### `com.apple.private.nullfs_allow`

इस अनुमति की अनुमति देती है कि एक nullfs फ़ाइल सिस्टम (डिफ़ॉल्ट रूप से निषेधित) माउंट किया जा सके। टूल: [**mount\_nullfs**](https://github.com/JamaicanMoose/mount\_nullfs/tree/master)।

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

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>
