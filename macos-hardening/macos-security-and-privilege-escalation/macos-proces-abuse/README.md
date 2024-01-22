# macOS प्रोसेस दुरुपयोग

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## MacOS प्रोसेस दुरुपयोग

MacOS, अन्य ऑपरेटिंग सिस्टम की तरह, **प्रोसेसेस के बीच इंटरैक्ट, कम्युनिकेट और डेटा शेयर करने** के लिए विभिन्न तरीके और तंत्र प्रदान करता है। जबकि ये तकनीकें सिस्टम के कुशल कार्यान्वयन के लिए आवश्यक हैं, इनका दुरुपयोग खतरा पैदा करने वाले लोग **दुर्भावनापूर्ण गतिविधियां करने** के लिए भी कर सकते हैं।

### लाइब्रेरी इंजेक्शन

लाइब्रेरी इंजेक्शन एक तकनीक है जिसमें हमलावर **एक प्रोसेस को दुर्भावनापूर्ण लाइब्रेरी लोड करने के लिए मजबूर करता है**। एक बार इंजेक्ट होने के बाद, लाइब्रेरी लक्षित प्रोसेस के संदर्भ में चलती है, जिससे हमलावर को प्रोसेस के समान अनुमतियां और पहुंच मिल जाती है।

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### फंक्शन हुकिंग

फंक्शन हुकिंग **सॉफ्टवेयर कोड के भीतर फंक्शन कॉल्स या मैसेजेस को इंटरसेप्ट करने** की प्रक्रिया है। फंक्शन्स को हुक करके, हमलावर प्रोसेस के व्यवहार को **मोडिफाई कर सकता है**, संवेदनशील डेटा को देख सकता है, या यहां तक कि निष्पादन प्रवाह पर नियंत्रण प्राप्त कर सकता है।

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### इंटर प्रोसेस कम्युनिकेशन

इंटर प्रोसेस कम्युनिकेशन (IPC) विभिन्न तरीकों को संदर्भित करता है जिसके द्वारा अलग-अलग प्रोसेसेस **डेटा शेयर और एक्सचेंज करते हैं**। जबकि IPC कई वैध अनुप्रयोगों के लिए मौलिक है, इसका दुरुपयोग प्रोसेस आइसोलेशन को उलटने, संवेदनशील जानकारी को लीक करने, या अनधिकृत क्रियाएं करने के लिए भी किया जा सकता है।

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### इलेक्ट्रॉन एप्लिकेशन्स इंजेक्शन

विशिष्ट env वेरिएबल्स के साथ निष्पादित इलेक्ट्रॉन एप्लिकेशन्स प्रोसेस इंजेक्शन के लिए संवेदनशील हो सकते हैं:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### डर्टी NIB

NIB फाइलें **यूजर इंटरफेस (UI) तत्वों** और एक एप्लिकेशन के भीतर उनके इंटरैक्शन्स को परिभाषित करती हैं। हालांकि, वे **मनमाने कमांड्स को निष्पादित कर सकती हैं** और **Gatekeeper एक बार निष्पादित हो चुके एप्लिकेशन को रोक नहीं पाता** अगर **NIB फाइल को संशोधित किया गया हो**। इसलिए, वे मनमाने प्रोग्राम्स को मनमाने कमांड्स निष्पादित करने के लिए उपयोग की जा सकती हैं:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### जावा एप्लिकेशन्स इंजेक्शन

जावा की कुछ क्षमताओं (जैसे कि **`_JAVA_OPTS`** env वेरिएबल) का दुरुपयोग करके जावा एप्लिकेशन को **मनमाने कोड/कमांड्स निष्पादित करने** के लिए बनाया जा सकता है।

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### .Net एप्लिकेशन्स इंजेक्शन

.Net एप्लिकेशन्स में कोड इंजेक्ट करना संभव है **.Net डिबगिंग फंक्शनलिटी का दुरुपयोग करके** (macOS सुरक्षा जैसे रनटाइम हार्डनिंग द्वारा संरक्षित नहीं)।

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### पर्ल इंजेक्शन

एक पर्ल स्क्रिप्ट को मनमाने कोड निष्पादित करने के विभिन्न विकल्पों की जांच करें:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### रूबी इंजेक्शन

रूबी env वेरिएबल्स का दुरुपयोग करके मनमाने स्क्रिप्ट्स को मनमाने कोड निष्पादित करना भी संभव है:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### पायथन इंजेक्शन

यदि पर्यावरण वेरिएबल **`PYTHONINSPECT`** सेट है, तो पायथन प्रोसेस एक बार समाप्त होने के बाद पायथन cli में ड्रॉप हो जाएगा। **`PYTHONSTARTUP`** का उपयोग एक पायथन स्क्रिप्ट को इंटरैक्टिव सत्र की शुरुआत में निष्पादित करने के लिए भी किया जा सकता है।\
हालांकि, ध्यान दें कि **`PYTHONSTARTUP`** स्क्रिप्ट **`PYTHONINSPECT`** द्वारा बनाए गए इंटरैक्टिव सत्र में निष्पादित नहीं होगी।

अन्य env वेरिएबल्स जैसे कि **`PYTHONPATH`** और **`PYTHONHOME`** भी पायथन कमांड को मनमाने कोड निष्पादित करने के लिए उपयोगी हो सकते हैं।

ध्यान दें कि **`pyinstaller`** के साथ संकलित निष्पादन योग्य फाइलें इन पर्यावरणीय वेरिएबल्स का उपयोग नहीं करेंगी भले ही वे एक एम्बेडेड पायथन का उपयोग कर रहे हों।

{% hint style="danger" %}
कुल मिलाकर मैंने पायथन को पर्यावरणीय वेरिएबल्स का दुरुपयोग करके मनमाने कोड निष्पादित करने का कोई तरीका नहीं पाया।\
हालांकि, अधिकांश लोग **Hombrew** का उपयोग करके पायथन को इंस्टॉल करते हैं, जो पायथन को डिफॉल्ट एडमिन यूजर के लिए एक **लिखने योग्य स्थान** पर इंस्टॉल करेगा। आप इसे कुछ इस तरह से हाइजैक कर सकते हैं:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
```markdown
यहां तक कि **root** भी इस कोड को चलाएगा जब python चल रहा हो।
{% endhint %}

## पहचान

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) एक ओपन सोर्स एप्लिकेशन है जो **प्रोसेस इंजेक्शन** क्रियाओं का पता लगा सकता है और उन्हें ब्लॉक कर सकता है:

* **Environmental Variables** का उपयोग करके: यह निम्नलिखित पर्यावरणीय चरों की उपस्थिति की निगरानी करेगा: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** और **`ELECTRON_RUN_AS_NODE`**
* **`task_for_pid`** कॉल्स का उपयोग करके: यह जब एक प्रोसेस दूसरे के **task port** को प्राप्त करना चाहता है तो उसे खोजने के लिए, जिससे प्रोसेस में कोड इंजेक्ट किया जा सकता है।
* **Electron apps params**: कोई भी **`--inspect`**, **`--inspect-brk`** और **`--remote-debugging-port`** कमांड लाइन आर्ग्युमेंट का उपयोग करके एक Electron एप्प को डिबगिंग मोड में शुरू कर सकता है, और इस तरह उसमें कोड इंजेक्ट कर सकता है।
* **symlinks** या **hardlinks** का उपयोग करके: आमतौर पर सबसे आम दुरुपयोग यह है कि **हमारे यूजर विशेषाधिकारों के साथ एक लिंक रखें**, और उसे **उच्च विशेषाधिकार** स्थान की ओर इंगित करें। हार्डलिंक और सिम्लिंक्स दोनों के लिए पता लगाना बहुत सरल है। यदि लिंक बनाने वाले प्रोसेस का **विशेषाधिकार स्तर** लक्ष्य फाइल से **अलग** है, तो हम एक **अलर्ट** बनाते हैं। दुर्भाग्यवश सिम्लिंक्स के मामले में ब्लॉकिंग संभव नहीं है, क्योंकि हमें लिंक के गंतव्य के बारे में जानकारी नहीं होती है निर्माण से पहले। यह Apple के EndpointSecuriy फ्रेमवर्क की एक सीमा है।

### अन्य प्रोसेस द्वारा किए गए कॉल्स

[**इस ब्लॉग पोस्ट**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) में आप पा सकते हैं कि कैसे **`task_name_for_pid`** फंक्शन का उपयोग करके अन्य **प्रोसेस द्वारा प्रोसेस में कोड इंजेक्ट करने** के बारे में जानकारी प्राप्त की जा सकती है और फिर उस अन्य प्रोसेस के बारे में जानकारी प्राप्त की जा सकती है।

नोट करें कि उस फंक्शन को कॉल करने के लिए आपको **वही uid** होना चाहिए जो प्रोसेस चला रहा है या **root** (और यह प्रोसेस के बारे में जानकारी लौटाता है, कोड इंजेक्ट करने का तरीका नहीं)।

## संदर्भ

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**official PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें।**
* **HackTricks** के लिए PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें। [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
```
