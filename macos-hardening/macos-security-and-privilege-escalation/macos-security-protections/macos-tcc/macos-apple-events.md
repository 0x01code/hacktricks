# macOS Apple Events

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## मूल जानकारी

**Apple Events** Apple के macOS में एक सुविधा है जो एप्लिकेशनों को एक दूसरे के साथ संवाद करने की अनुमति देती है। ये **Apple Event Manager** का हिस्सा है, जो macOS ऑपरेटिंग सिस्टम का एक घटक है जो इंटरप्रोसेस कम्यूनिकेशन को संभालने के लिए जिम्मेदार है। यह सिस्टम एक एप्लिकेशन को दूसरे एप्लिकेशन को एक विशेष कार्य करने का अनुरोध करने के लिए संदेश भेजने के लिए एक एप्लिकेशन को सक्षम करता है, जैसे कि एक फ़ाइल खोलना, डेटा प्राप्त करना, या एक कमांड को निष्पादित करना।

मीना डेमन `/System/Library/CoreServices/appleeventsd` है जो सेवा `com.apple.coreservices.appleevents` को रजिस्टर करता है।

हर एप्लिकेशन जो इवेंट प्राप्त कर सकता है, वह इस डेमन के साथ अपना Apple Event Mach Port जांचेगा। और जब एक ऐप इसे इवेंट भेजना चाहता है, तो ऐप डेमन से इस पोर्ट का अनुरोध करेगा।

Sandboxed एप्लिकेशन को इवेंट भेजने की अनुमति के लिए विशेषाधिकार जैसे `allow appleevent-send` और `(allow mach-lookup (global-name "com.apple.coreservices.appleevents))` की आवश्यकता होती है। ध्यान दें कि entitlements जैसे `com.apple.security.temporary-exception.apple-events` उन्हें प्रत्यक्ष इवेंट भेजने की पहुंच किसके पास है इसे प्रतिबंधित कर सकते हैं जिसके लिए entitlements जैसे `com.apple.private.appleevents` की आवश्यकता होगी।

{% hint style="success" %}
यह संदेश भेजने के बारे में जानकारी लॉग करने के लिए एनवी वेरिएबल **`AEDebugSends`** का उपयोग संभव है:
```bash
AEDebugSends=1 osascript -e 'tell application "iTerm" to activate'
```
{% endhint %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
