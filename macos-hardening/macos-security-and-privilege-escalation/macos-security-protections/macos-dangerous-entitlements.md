# macOS Dangerous Entitlements & TCC perms

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, **The PEASS Family** की खोज करें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

{% hint style="warning" %}
ध्यान दें कि **`com.apple`** से शुरू होने वाली अधिकार तीसरे पक्षों के लिए उपलब्ध नहीं हैं, केवल Apple उन्हें प्रदान कर सकता है।
{% endhint %}

## उच्च

### `com.apple.rootless.install.heritable`

अधिकार **`com.apple.rootless.install.heritable`** **SIP को छलना** करने की अनुमति देता है। [इसके लिए अधिक जानकारी के लिए देखें](macos-sip.md#com.apple.rootless.install.heritable)।

### **`com.apple.rootless.install`**

अधिकार **`com.apple.rootless.install`** **SIP को छलना** करने की अनुमति देता है। [इसके लिए अधिक जानकारी के लिए देखें](macos-sip.md#com.apple.rootless.install)।

### **`com.apple.system-task-ports` (पहले `task_for_pid-allow` कहा गया)**

यह अधिकार किसी भी **प्रक्रिया के लिए कार्य पोर्ट प्राप्त करने की** अनुमति देता है, केवल कर्णेल को छोड़कर। [**इसके लिए अधिक जानकारी के लिए देखें**](../macos-proces-abuse/macos-ipc-inter-process-communication/)।

### `com.apple.security.get-task-allow`

यह अधिकार अन्य प्रक्रियाओं को **`com.apple.security.cs.debugger`** अधिकार के साथ इस अधिकार वाले बाइनरी द्वारा चलाई गई प्रक्रिया का कार्य पोर्ट प्राप्त करने और उस पर कोड इंजेक्शन करने की अनुमति देता है। [**इसके लिए अधिक जानकारी के लिए देखें**](../macos-proces-abuse/macos-ipc-inter-process-communication/)।

### `com.apple.security.cs.debugger`

डेबगिंग टूल अधिकार वाले ऐप्स `task_for_pid()` को कॉल कर सकते हैं ताकि वे `Get Task Allow` अधिकार को `true` पर सेट करके असाइन किए गए और तीसरे पक्ष के ऐप्स के लिए मान्य कार्य पोर्ट प्राप्त कर सकें। हालांकि, डेबगिंग टूल अधिकार के साथ, एक डीबगर **प्रक्रियाओं के कार्य पोर्ट प्राप्त नहीं कर सकता** है जो **`Get Task Allow` अधिकार नहीं हैं**, और जिनकी रक्षा कर्णेल अखंडता सुरक्षा द्वारा की गई है। [**इसके लिए अधिक जानकारी के लिए देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger)।

### `com.apple.security.cs.disable-library-validation`

यह अधिकार \*\*ऐप्प्स को फ्रेमवर्क, प्लगइन, या लाइब्रेरी लोड करने की अनुमति देता है जो न तो Apple द्वारा साइन किए गए हों और न ही मुख्य एग्जीक्यूटेबल के साथ साइन किए गए टीम आईडी के साथ साइन किए गए हों, इसलिए एक हमलावर किसी भी अर्बिट्रे लाइब्रेरी लोड को कोड इंजेक्शन करने के लिए उपयोग कर सकता है। [**इसके लिए अधिक जानकारी के लिए देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation)।

### `com.apple.private.security.clear-library-validation`

यह अधिकार **`com.apple.security.cs.disable-library-validation`** के बहुत ही समान है, **सीधे लाइब्रेरी मान्यता को अक्षम करने की बजाय**, यह प्रक्रिया को इसे अक्षम करने के लिए `csops` सिस्टम कॉल करने की अनुमति देता है।\
[**इसके लिए अधिक जानकारी के लिए देखें**](https://theevilbit.github.io/posts/com.apple.private.security.clear-library-validation/)।

### `com.apple.security.cs.allow-dyld-environment-variables`

यह अधिकार **DYLD environment variables** का उपयोग करने की अनुमति देता है जो लाइब्रेरी और कोड इंजेक्शन के लिए उपयोग किया जा सकता है। [**इसके लिए अधिक जानकारी के लिए देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables)।

### `com.apple.private.tcc.manager` या `com.apple.rootless.storage`.`TCC`

[**इस ब्लॉग के अनुसार**](https://objective-see.org/blog/blog\_0x4C.html) **और** [**इस ब्लॉग के अनुसार**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/), ये अधिकार **TCC** डेटाबेस को **संशोधित** करने की अनुमति देते हैं।

### **`system.install.apple-software`** और **`system.install.apple-software.standar-user`**

ये अधिकार उपयोगकर्ता से अनुमति मांगे बिना **सॉफ्टवेयर स्थापित करने की** अनुमति देते हैं, जो **एक विशेषाधिकार उन्नयन** के लिए सहायक हो सकता है।

### `com.apple.private.security.kext-management`

कर्णेल से **कर्णेल एक्सटेंशन लोड करने के लिए** अधिकार आवश्यक है।

### **`com.apple.private.icloud-account-access`**

अधिकार **`com.apple.private.icloud-account-access`** संभावित है **`com.apple.iCloudHelper`** XPC सेवा के साथ संवाद करने की जिससे **iCloud टोकन प्रदान** किए जा सकते हैं।

**iMovie** और **Garageband** के पास इस अधिकार था।

उस अधिकार से iCloud टोकन प्राप्त करने के लिए उस अधिकार से अधिक जानकारी के लिए टॉक देखें: [**#OBTS v5.0: "What Happens on your Mac, Stays on Apple's iCloud?!" - Wojciech Regula**](https://www.youtube.com/watch?v=\_6e2LhmxVc0)

### `com.apple.private.tcc.manager.check-by-audit-token`

TODO: मुझे यह नहीं पता कि यह क्या करने की अनुमति देता है

### `com.apple.private.apfs.revert-to-snapshot`

TODO: [**इस रिपोर्ट में**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **उल्लेख किया गया है कि इसका उपयोग** एक बूट के बाद SSV संरक्षित सामग्री को अपडेट करने के लिए किया जा सकता है। यदि आप जानते हैं कि यह कैसे करें तो कृपया एक पीआर भेजें!

### `com.apple.private.apfs.create-sealed-snapshot`

TODO: [**इस रिपोर्ट में**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/) **उल्लेख किया गया है कि इसका उपयोग** एक बूट के बाद SSV संरक्षित सामग्री को अपडेट करने के लिए किया जा सकता है। यदि आप जानते हैं कि यह कैसे करें तो कृपया एक पीआर भेजें!

### `keychain-access-groups`

यह अधिकार सूची **कीचेन** समूहों को दर्शाता है जिनका एप्लिकेशन तक पहुंच है:

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

फुल डिस्क एक्सेस अनुमतियाँ देता है, जो TCC की सबसे उच्च अनुमतियों में से एक है जो आपके पास हो सकती है।

### **`kTCCServiceAppleEvents`**

ऐप्लिकेशन को अन्य ऐप्लिकेशनों को इवेंट भेजने की अनुमति देता है जो सामान्यत: कार्यों को स्वचालित करने के लिए उपयोग किए जाते हैं। अन्य ऐप्स को नियंत्रित करके, यह उन अन्य ऐप्स को दी गई अनुमतियों का दुरुपयोग कर सकता है।

जैसे कि उन्हें उपयोगकर्ता से उसका पासवर्ड मांगने के लिए कहना:

```bash
osascript -e 'tell app "App Store" to activate' -e 'tell app "App Store" to activate' -e 'tell app "App Store" to display dialog "App Store requires your password to continue." & return & return default answer "" with icon 1 with hidden answer with title "App Store Alert"'
```

या उन्हें **मनमाने कार्रवाई** करने के लिए।

### **`kTCCServiceEndpointSecurityClient`**

इसमें, अन्य अनुमतियों के बीच, **उपयोगकर्ताओं के TCC डेटाबेस में लिखने** की अनुमति है।

### **`kTCCServiceSystemPolicySysAdminFiles`**

इसे **उपयोगकर्ता के `NFSHomeDirectory`** विशेषता को **बदलने** की अनुमति है जो उसके होम फोल्डर पथ को बदल देती है और इसलिए **TCC को छलना** अनुमति देती है।

### **`kTCCServiceSystemPolicyAppBundles`**

ऐप्स बंडल (ऐप.ऐप के अंदर) के भीतर फ़ाइलों को संशोधित करने की अनुमति देती है, जो **डिफ़ॉल्ट रूप से निषिद्ध** है।

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

यह जांचना संभव है कि _सिस्टम सेटिंग्स_ > _गोपनीयता और सुरक्षा_ > _ऐप प्रबंधन_ में किसके पास यह पहुंच है।

### `kTCCServiceAccessibility`

प्रक्रिया **macOS पहुंचनीयता सुविधाओं का दुरुपयोग** कर सकेगी, जिसका मतलब है कि उदाहरण के लिए वह कीबोर्ड कुंजियों को दबा सकेगी। इसलिए वह Finder जैसे ऐप को नियंत्रित करने के लिए पहुंच का अनुरोध कर सकता है और इस अनुमति के साथ स्वीकृति दे सकता है।

## मध्यम

### `com.apple.security.cs.allow-jit`

यह अधिकार **लिखने और क्रियाशील मेमोरी बनाने** की अनुमति देता है जिसे `mmap()` सिस्टम कार्य को `MAP_JIT` झंडा पार करके। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit)।

### `com.apple.security.cs.allow-unsigned-executable-memory`

यह अधिकार **C कोड को ओवरराइड या पैच करने** की अनुमति देता है, लंबे समय से विरोधित **`NSCreateObjectFileImageFromMemory`** का उपयोग करने के लिए (जो मौलिक रूप से असुरक्षित है), या **DVDPlayback** फ़्रेमवर्क का उपयोग करने के लिए। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory)।

{% hint style="danger" %}
इस अधिकार को शामिल करने से आपका ऐप मेमोरी-असुरक्षित कोड भाषाओं में सामान्य सुरक्षा दोषों का सामना कर सकता है। ध्यान से विचार करें कि क्या आपके ऐप को यह अपवाद आवश्यक है।
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

यह अधिकार अपनी खुद की एक्जीक्यूटेबल फ़ाइलों के खंडों को **बदलने** की अनुमति देता है ताकि ज़बरदस्ती से बाहर निकल सके। अधिक जानकारी के लिए [**यह देखें**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection)।

डिसेबल एक्जीक्यूटेबल मेमोरी सुरक्षा अधिकार एक अत्यधिक अधिकार है जो आपके ऐप से एक मौलिक सुरक्षा सुरक्षा को हटा देता है, जिससे हमलावर को आपके ऐप के एक्जीक्यूटेबल कोड को पुनः लिखने की संभावना होती है बिना किसी पहचान के। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। संभावना होती है। स \`\`\` \[Key] com.apple.private.tcc.allow-prompting \[Value] \[Array] \[String] kTCCServiceAll \`\`\` ### \*\*\`kTCCServicePostEvent\`\*\*

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर फॉलो\*\* करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके।

</details>
