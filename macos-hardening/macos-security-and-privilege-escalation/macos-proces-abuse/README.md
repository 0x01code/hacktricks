# macOS प्रक्रिया दुरुपयोग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## MacOS प्रक्रिया दुरुपयोग

MacOS, जैसे कि किसी भी ऑपरेटिंग सिस्टम, **प्रक्रियाओं को इंटरैक्ट, संचार करने और डेटा साझा करने** के लिए विभिन्न तकनीक और तंत्र प्रदान करता है। ये तकनीक अच्छे सिस्टम कार्य के लिए आवश्यक हैं, लेकिन इन्हें धोखाधड़ीकरण करने वाले अभियांताओं द्वारा **दुरुपयोग किया जा सकता है**।

### लाइब्रेरी इंजेक्शन

लाइब्रेरी इंजेक्शन एक तकनीक है जिसमें एक हमलावर **प्रक्रिया को एक दुरुपयोगी लाइब्रेरी लोड करने के लिए मजबूर करता है**। एक बार इंजेक्ट किया गया, लाइब्रेरी लक्ष्य प्रक्रिया के संदर्भ में चलती है, हमलावर को प्रक्रिया के साथ समान अनुमतियों और एक्सेस प्रदान करती है।

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### फंक्शन हुकिंग

फंक्शन हुकिंग में **सॉफ्टवेयर कोड के भीतर फंक्शन कॉल्स** या संदेशों को अंतर्गत करना शामिल है। फंक्शन हुकिंग के द्वारा, एक हमलावर **प्रक्रिया के व्यवहार को संशोधित** कर सकता है, संवेदनशील डेटा को देख सकता है, या नियंत्रण प्राप्त कर सकता है प्रयोग की प्रवाह।

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### इंटर प्रोसेस कम्यूनिकेशन

इंटर प्रोसेस कम्यूनिकेशन (IPC) अलग-अलग प्रक्रियाओं द्वारा **डेटा साझा और विनिमय** करने के विभिन्न तरीकों को संदर्भित करता है। IPC बहुत से वैध एप्लिकेशनों के लिए मौलिक है, लेकिन इसका दुरुपयोग प्रक्रिया अलगाव, संवेदनशील जानकारी लीक, या अनधिकृत क्रियाएँ करने के लिए किया जा सकता है।

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### इलेक्ट्रॉन एप्लिकेशन इंजेक्शन

निश्चित एनवायरनमेंट वेरिएबल्स के साथ निष्क्रिय किए गए इलेक्ट्रॉन एप्लिकेशन इंजेक्शन के लिए संभावना है:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### क्रोमियम इंजेक्शन

`--load-extension` और `--use-fake-ui-for-media-stream` फ्लैग्स का उपयोग करके **एक मैन इन द ब्राउज़र हमला** करना संभव है, जिससे कीस्ट्रोक्स, ट्रैफिक, कुकीज़ चुराया जा सकता है, पेजों में स्क्रिप्ट इंजेक्ट किया जा सकता है...:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### डर्टी NIB

NIB फ़ाइलें **एप्लिकेशन के भीतर यूज़र इंटरफ़ेस (UI) तत्वों** और उनके बीच के इंटरैक्शन को परिभाषित करती हैं। हालांकि, वे **विचारात्मक आदेश चला सकती हैं** और **गेटकीपर नहीं रोकता** अगर एक **NIB फ़ाइल संशोधित किया गया है** तो पहले से ही चल रहे एप्लिकेशन को चलाने से। इसलिए, इन्हें इस्तेमाल किया जा सकता है विचारात्मक कार्यक्रमों को विचारात्मक कार्यक्रमों को चलाने के लिए:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### जावा एप्लिकेशन इंजेक्शन

कुछ जावा क्षमताओं (जैसे **`_JAVA_OPTS`** एनवायरनमेंट वेरिएबल) का दुरुपयोग करके एक जावा एप्लिकेशन को **विचारात्मक कोड/आदेश चलाने** के लिए दुरुपयोग किया जा सकता है।

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### .Net एप्लिकेशन इंजेक्शन

मैकओएस संरक्षणों जैसे रनटाइम हार्डनिंग जैसे संरक्षणों द्वारा संरक्षित नहीं है, .Net एप्लिकेशन में कोड इंजेक्शन किया जा सकता है **.Net डीबगिंग क्षमता का दुरुपयोग करके**।

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### पर्ल इंजेक्शन

पर्ल स्क्रिप्ट में विचारात्मक कोड चलाने के लिए विभिन्न विकल्पों की जांच करें:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### रूबी इंजेक्शन

रूबी एनवायरनमेंट वेरिएबल्स का दुरुपयोग करके विचारात्मक स्क्रिप्ट्स को विचारात्मक कोड चलाने के लिए संभव है:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### पायथन इंजेक्शन

यदि एनवायरनमेंट वेरिएबल **`PYTHONINSPECT`** सेट किया गया है, तो पायथन प्रक्रिया एक पायथन cli में गिर जाएगी जब यह समाप्त होती है। एक इंटरैक्टिव सत्र की शुरुआत में एक पायथन स्क्रिप्ट को चलाने के लिए **`PYTHONSTARTUP`** का उपयोग भी संभव है।\
हालांकि, ध्यान दें कि **`PYTHONSTARTUP`** स्क्रिप्ट **`PYTHONINSPECT`** इंटरैक्टिव सत्र बनाते समय नहीं चलाया जाएगा।

**`PYTHONPATH`** और **`PYTHONHOME`** जैसे अन्य एनवायरनमेंट वेरिएबल्स भी एक पायथन कमांड को विचारात्मक कोड चलाने के लिए उपयोगी हो सकते हैं।

ध्यान दें कि **`pyinstaller`** के साथ कंपाइल किए गए एक्जीक्यूटेबल्स इन एनवायरनमेंट वेरिएबल्स का उपयोग नहीं करेंगे यदि वे एक एम्बेडेड पायथन का उपयोग करके चल रहे हैं।

{% hint style="danger" %}
समग्र रूप से मैंने पायथन को विचारात्मक कोड चलाने का कोई तरीका नहीं पाया।\
हालांकि, अधिकांश लो
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
## डिटेक्शन

### ढाल

[**ढाल**](https://theevilbit.github.io/shield/) ([**गिटहब**](https://github.com/theevilbit/Shield)) एक ओपन सोर्स एप्लिकेशन है जो **प्रक्रिया इंजेक्शन की पहचान और ब्लॉक** कर सकता है:

* **पर्यावरणीय चर** का उपयोग: यह निम्नलिखित पर्यावरणीय चरों की उपस्थिति का मॉनिटर करेगा: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** और **`ELECTRON_RUN_AS_NODE`**
* **`task_for_pid`** कॉल का उपयोग: एक प्रक्रिया को दूसरे की **टास्क पोर्ट** प्राप्त करना चाहती है जिससे प्रक्रिया में कोड इंजेक्ट किया जा सके।
* **इलेक्ट्रॉन ऐप्स पैरामीटर**: कोई **`--inspect`**, **`--inspect-brk`** और **`--remote-debugging-port`** कमांड लाइन आर्ग्यूमेंट का उपयोग करके एक इलेक्ट्रॉन ऐप को डीबगिंग मोड में शुरू कर सकता है, और इसे इंजेक्ट कर सकता है।
* **सिमलिंक्स** या **हार्डलिंक्स** का उपयोग: सामान्यत: सबसे आम दुरुपयोग हमारे उपयोगकर्ता विशेषाधिकारों के साथ एक लिंक रखना है, और उच्च विशेषाधिकार स्थान की ओर इसे पॉइंट करना है। हार्डलिंक और सिमलिंक्स के लिए डिटेक्शन बहुत सरल है। अगर लिंक बनाने वाली प्रक्रिया का **लक्ष्य फ़ाइल से विभिन्न विशेषाधिकार स्तर** है, तो हम एक **अलर्ट** बनाते हैं। दुर्भाग्यवश: सिमलिंक्स के मामले में ब्लॉकिंग संभव नहीं है, क्योंकि हमें लिंक के गंतव्य के बारे में जानकारी नहीं है पहले से। यह Apple के EndpointSecuriy framework की एक सीमा है।

### अन्य प्रक्रियाओं द्वारा की गई कॉल

[**इस ब्लॉग पोस्ट**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) में आप देख सकते हैं कि कैसे फ़ंक्शन **`task_name_for_pid`** का उपयोग करके एक प्रक्रिया में कोड इंजेक्शन करने वाली अन्य **प्रक्रियाओं के बारे में जानकारी प्राप्त** की जा सकती है और फिर उस अन्य प्रक्रिया के बारे में जानकारी प्राप्त की जा सकती है।

ध्यान दें कि उस फ़ंक्शन को कॉल करने के लिए आपको प्रक्रिया चलाने वाले व्यक्ति के **यूआईडी** के समान होना चाहिए या **रूट** (और यह प्रक्रिया के बारे में जानकारी देता है, कोड इंजेक्शन का एक तरीका नहीं)।

## संदर्भ

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)
