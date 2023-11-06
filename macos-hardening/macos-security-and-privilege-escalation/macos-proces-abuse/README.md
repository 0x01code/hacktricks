# macOS प्रक्रिया दुरुपयोग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

## MacOS प्रक्रिया दुरुपयोग

MacOS, किसी भी अन्य ऑपरेटिंग सिस्टम की तरह, **प्रक्रियाओं को इंटरैक्ट, संचार और डेटा साझा करने** के लिए विभिन्न तकनीकों और यंत्रों की प्रदान करता है। यद्यपि ये तकनीक दक्ष सिस्टम कार्य के लिए आवश्यक हैं, लेकिन इनका दुरुपयोग भी किया जा सकता है ताकि आपत्तिजनक गतिविधियों को **क्रियान्वित** किया जा सके।

### लाइब्रेरी इंजेक्शन

लाइब्रेरी इंजेक्शन एक तकनीक है जिसमें एक हमलावर **प्रक्रिया को एक आपत्तिजनक लाइब्रेरी लोड करने के लिए मजबूर किया जाता है**। एक बार इंजेक्शन हो जाने पर, लाइब्रेरी लक्ष्य प्रक्रिया के संदर्भ में चलती है, जिससे हमलावर को प्रक्रिया के साथी अनुमतियों और पहुंच की प्राप्ति होती है।

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### फंक्शन हुकिंग

फंक्शन हुकिंग में, सॉफ्टवेयर कोड के भीतर **फंक्शन कॉल** या **संदेशों को अवरोधित करना** शामिल होता है। फंक्शन हुकिंग के द्वारा, एक हमलावर प्रक्रिया के व्यवहार को **संशोधित** कर सकता है, संवेदनशील डेटा का अवलोकन कर सकता है, या यहां तक कि क्रियान्वयन धारण कर सकता है।

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### इंटर प्रक्रिया संचार

इंटर प्रक्रिया संचार (IPC) अलग-अलग तकनीकों को संदर्भित करता है, जिनसे अलग-अलग प्रक्रियाएं **डेटा साझा और आपस में विनिमय करती हैं**। IPC बहुत सारे वैध अनुप्रयोगों के लिए मूल्यांकन है, लेकिन इसका दुरुपयोग भी किया जा सकता है ताकि प्रक्रिया अलगाव को उल्लंघन कर सके, संवेदनशील जानकारी छिड़काव कर सके, या अनधिकृत कार्रवाई कर सके।

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### इलेक्ट्रॉन एप्लिकेशन इंजेक्शन

निश्चित env variables के साथ निष्पादित इलेक्ट्रॉन एप्लिकेशन प्रक्रिया इंजेक्शन के प्रति संवेदनशील हो सकती हैं:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### गंदी NIB

NIB फ़ाइलें **उपयोगकर्ता इंटरफ़ेस (UI) तत्वों** को परिभाषित करती हैं और उनके अन्तर्घटनों को विवरणित कर
### अन्य प्रक्रियाओं द्वारा किए गए कॉल

[**इस ब्लॉग पोस्ट**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) में आप देख सकते हैं कि कैसे फ़ंक्शन **`task_name_for_pid`** का उपयोग करके एक प्रक्रिया में कोड इंजेक्शन करने वाली अन्य **प्रक्रियाओं के बारे में जानकारी प्राप्त** की जा सकती है और फिर उस अन्य प्रक्रिया के बारे में जानकारी प्राप्त की जा सकती है।

ध्यान दें कि इस फ़ंक्शन को कॉल करने के लिए आपको प्रक्रिया चलाने वाले उसी uid के रूप में होना चाहिए या **रूट** होना चाहिए (और यह प्रक्रिया के बारे में जानकारी देता है, कोड इंजेक्शन करने का एक तरीका नहीं है)।

## संदर्भ

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में जमा करके**।

</details>
