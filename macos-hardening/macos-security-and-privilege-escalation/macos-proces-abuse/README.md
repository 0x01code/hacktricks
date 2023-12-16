# macOS प्रक्रिया दुरुपयोग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह खोजें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

## MacOS प्रक्रिया दुरुपयोग

MacOS, किसी भी अन्य ऑपरेटिंग सिस्टम की तरह, **प्रक्रियाओं को इंटरैक्ट, संचार और डेटा साझा करने** के लिए विभिन्न तकनीकों और यंत्रों की प्रदान करता है। ये तकनीक दक्ष प्रणाली के सही कार्य के लिए आवश्यक होती हैं, लेकिन इनका दुरुपयोग भी खतरनाक गतिविधियों को करने के लिए थ्रेट एक्टर्स द्वारा किया जा सकता है।

### लाइब्रेरी इंजेक्शन

लाइब्रेरी इंजेक्शन एक तकनीक है जिसमें एक हमलावर्धक **प्रक्रिया को एक खतरनाक लाइब्रेरी लोड करने के लिए मजबूर किया जाता है**। इंजेक्शन के बाद, लाइब्रेरी लक्ष्य प्रक्रिया के संदर्भ में चलती है, हमलावर्धक को प्रक्रिया के साथ एक ही अनुमतियों और पहुंच की प्रदान करती है।

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### फंक्शन हुकिंग

फंक्शन हुकिंग में, सॉफ्टवेयर कोड के भीतर **फंक्शन कॉल** या **संदेशों को अवरोधित करना** शामिल होता है। फंक्शन हुकिंग के द्वारा, एक हमलावर्धक प्रक्रिया का व्यवहार **संशोधित** कर सकता है, संवेदनशील डेटा का अवलोकन कर सकता है, या नियंत्रण प्राप्त कर सकता है।

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### इंटर प्रक्रिया संचार

इंटर प्रक्रिया संचार (IPC) अलग-अलग तकनीकों को संदर्भित करता है जिनसे अलग-अलग प्रक्रियाएं **डेटा साझा और आपस में विनिमय करती हैं**। IPC बहुत सारे वैधानिक अनुप्रयोगों के लिए मूल्यांकन है, लेकिन इसका दुरुपयोग प्रक्रिया अलगाव को अवरुद्ध करने, संवेदनशील जानकारी लीक करने या अनधिकृत कार्रवाई करने के लिए किया जा सकता है।

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### इलेक्ट्रॉन एप्लिकेशन इंजेक्शन

निश्चित env variables के साथ निष्पादित इलेक्ट्रॉन एप्लिकेशन प्रक्रिया इंजेक्शन के लिए संकटग्रस्त हो सकते हैं:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### गंदी NIB

NIB फ़ाइलें **उपयोगकर्ता इंटरफ़ेस (UI) तत्वों** को परिभाषित करती हैं और एक एप्लिकेशन के भीतर उनके संवेदनशीलता के साथ इंटरैक्शन करती हैं। हालांकि, वे **अनियंत्रित आदेशों को निष्पादित कर सकते
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
यह कोड चलाने पर **रूट** भी इस कोड को चलाएगा।
{% endhint %}

## पता लगाना

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) एक ओपन सोर्स एप्लिकेशन है जो **प्रक्रिया इंजेक्शन की पहचान और रोकथाम** कर सकता है:

* **पर्यावरणीय चरों** का उपयोग करके: यह निम्नलिखित पर्यावरणीय चरों **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** और **`ELECTRON_RUN_AS_NODE`** की मौजूदगी का मॉनिटरिंग करेगा।
* **`task_for_pid`** कॉल का उपयोग करके: जब एक प्रक्रिया दूसरी प्रक्रिया में कोड इंजेक्शन करने की इच्छा रखती है, तो यह उस दूसरी प्रक्रिया के टास्क पोर्ट प्राप्त करने की पहचान करेगा।
* **Electron ऐप्स पैरामीटर**: कोई व्यक्ति इलेक्ट्रॉन ऐप को डिबगिंग मोड में शुरू करने के लिए **`--inspect`**, **`--inspect-brk`** और **`--remote-debugging-port`** कमांड लाइन तर्क का उपयोग कर सकता है, और इसलिए इसमें कोड इंजेक्शन कर सकता है।
* **सिमलिंक्स** या **हार्डलिंक्स** का उपयोग करके: सामान्यतः सबसे सामान्य दुरुपयोग यह है कि हम **अपने उपयोगकर्ता विशेषाधिकारों** के साथ एक लिंक रखते हैं, और उच्चतर विशेषाधिकार स्थान की ओर इसे पॉइंट करते हैं। हार्डलिंक और सिमलिंक्स के लिए पहचानना बहुत सरल है। यदि लिंक बनाने वाली प्रक्रिया का **विशेषाधिकार स्तर** लक्ष्य फ़ाइल से अलग होता है, तो हम एक **चेतावनी** बनाते हैं। दुर्भाग्य से सिमलिंक्स ब्लॉक करना संभव नहीं है, क्योंकि हमें लिंक के गंतव्य के बारे में जानकारी पहले से ही नहीं होती है। यह Apple के EndpointSecuriy framework की एक सीमा है।

### अन्य प्रक्रियाओं द्वारा किए गए कॉल

[**इस ब्लॉग पोस्ट**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) में आप देख सकते हैं कि कैसे फ़ंक्शन **`task_name_for_pid`** का उपयोग करके एक प्रक्रिया में कोड इंजेक्शन करने वाली अन्य **प्रक्रियाओं के बारे में जानकारी प्राप्त** की जा सकती है और फिर उस अन्य प्रक्रिया के बारे में जानकारी प्राप्त की जा सकती है।

ध्यान दें कि उस फ़ंक्शन को कॉल करने के लिए आपको प्रक्रिया चलाने वाले व्यक्ति के **एक ही uid** होना चाहिए या **रूट** होना चाहिए (और यह कोड इंजेक्शन का एक तरीका नहीं है, बल्कि इससे प्रक्रिया के बारे में जानकारी प्राप्त होती है)।

## संदर्भ

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं** या HackTricks को **PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **[💬](https://emojipedia.org/speech-balloon/) Discord समूह** या **[telegram समूह](https://t.me/peass)** में शामिल हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में।**

</details>
