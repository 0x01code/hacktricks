<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की उपलब्धता** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें**.

</details>


# हमले का सारांश

कल्पना करें कि एक सर्वर है जो कुछ ज्ञात साफ पाठ डेटा के बाद एक गुप्त जोड़कर उस डेटा को हैश करके उसे साइन कर रहा है। यदि आपको पता है:

* **गुप्त जोड़कर की लंबाई** (इसे एक दिए गए लंबाई सीमा से भी bruteforce किया जा सकता है)
* **साफ पाठ डेटा**
* **एल्गोरिदम (और यह हमले के लिए संकटग्रस्त है)**
* **पैडिंग ज्ञात है**
* आमतौर पर एक डिफ़ॉल्ट पैडिंग का उपयोग किया जाता है, इसलिए यदि अन्य 3 आवश्यकताएँ पूरी होती हैं, तो यह भी होती है
* पैडिंग गुप्त जोड़कर+डेटा की लंबाई पर निर्भर करता है, इसलिए गुप्त जोड़कर की लंबाई की आवश्यकता होती है

तो, एक **हमलावर** को **डेटा** को **जोड़ने** और **पिछले डेटा + जोड़ा गया डेटा** के लिए एक मान्य **हस्ताक्षर** उत्पन्न करने की संभावना होती है।

## कैसे?

मूल रूप से, संकटग्रस्त एल्गोरिदम डेटा के एक ब्लॉक को पहले से ही हैश करके हैश उत्पन्न करते हैं, और फिर, पहले से बनाए गए हैश (स्थिति) से अगले ब्लॉक डेटा को जोड़ते हैं और उसे हैश करते हैं।

फिर, कल्पित करें कि गुप्त जोड़कर "गुप्त" है और डेटा "डेटा" है, "गुप्तडेटा" का MD5 6036708eba0d11f6ef52ad44e8b74d5b है।\
यदि एक हमलावर "जोड़ना" शब्द को जोड़ना चाहता है, तो वह कर सकता है:

* 64 "A" का MD5 उत्पन्न करें
* पहले से शुरू किए गए हैश की स्थिति को 6036708eba0d11f6ef52ad44e8b74d5b पर बदलें
* शब्द "जोड़ना" को जोड़ें
* हैश को समाप्त करें और परिणामी हैश "गुप्त" + "डेटा" + "पैडिंग" + "जोड़ना" के लिए **मान्य होगा**

## **उपकरण**

{% embed url="https://github.com/iagox86/hash_extender" %}

# संदर्भ

आप इस हमले को अच्छी तरह से समझाने के लिए [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks) में देख सकते हैं


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की उपलब्धता** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospol
