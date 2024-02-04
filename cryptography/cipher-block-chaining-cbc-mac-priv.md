<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** को PRs जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


# CBC

यदि **कुकी** **केवल** **उपयोगकर्ता नाम** है (या कुकी का पहला हिस्सा उपयोगकर्ता नाम है) और आप उपयोगकर्ता "**व्यवस्थापक**" का अनुकरण करना चाहते हैं। तो, आप उपयोगकर्ता नाम **"बीडमिन"** बना सकते हैं और कुकी के **पहले बाइट** का **ब्रूटफोर्स** कर सकते हैं।

# CBC-MAC

रहस्यमयता में, एक **साइफर ब्लॉक चेनिंग संदेश प्रमाणीकरण कोड** (**CBC-MAC**) एक तकनीक है जिसका उपयोग एक ब्लॉक साइफर से संदेश प्रमाणीकरण कोड निर्मित करने के लिए किया जाता है। संदेश को कुछ ब्लॉक साइफर एल्गोरिदम के साथ सीबीसी मोड में एन्क्रिप्ट किया जाता है ताकि प्रत्येक ब्लॉक पिछले ब्लॉक के सही एन्क्रिप्शन पर निर्भर हो। यह आपसी आश्रितता सुनिश्चित करता है कि किसी भी प्लेनटेक्स्ट **बिट्स** में कोई **परिवर्तन** होने पर **अंतिम एन्क्रिप्टेड ब्लॉक** को एक ऐसे तरीके से **परिवर्तित** करेगा जिसे पूर्वानुमान या ब्लॉक साइफर कुंजी को नहीं जानकर नहीं रोका जा सकता।

संदेश m का CBC-MAC निर्धारित करने के लिए, किसी भी शून्य प्रारंभीक वेक्टर के साथ m को सीबीसी मोड में एन्क्रिप्ट किया जाता है और अंतिम ब्लॉक को रखा जाता है। निम्न चित्र में एक संदेश का CBC-MAC की गणना करने की रूपरेखा दी गई है![m\_{1}\\|m\_{2}\\|\cdots \\|m\_{x}](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) एक गुप्त कुंजी k और एक ब्लॉक साइफर E के साथ:

![CBC-MAC structure (en).svg](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# सुरक्षा दोष

CBC-MAC के साथ आम तौर पर **इस्तेमाल किया जाने वाला IV 0 होता है**।\
यह एक समस्या है क्योंकि 2 ज्ञात संदेश (`m1` और `m2`) स्वतंत्र रूप से 2 हस्ताक्षर (`s1` और `s2`) उत्पन्न करेंगे। इसलिए:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

फिर m1 और m2 को जोड़कर बनाया गया संदेश (m3) 2 हस्ताक्षर (s31 और s32) उत्पन्न करेगा:

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**जिसे एन्क्रिप्शन की कुंजी को नहीं जानते हुए गणना किया जा सकता है।**

सोचिए आप **8 बाइट** ब्लॉक में नाम **प्रशासक** को एन्क्रिप्ट कर रहे हैं:

* `Administ`
* `rator\00\00\00`

आप उपयोगकर्ता नाम **Administ** (m1) बना सकते हैं और हस्ताक्षर (s1) प्राप्त कर सकते हैं।\
फिर, आप `rator\00\00\00 XOR s1` के परिणाम को उपयोगकर्ता नाम बना सकते हैं। यह `E(m2 XOR s1 XOR 0)` उत्पन्न करेगा जो s32 है।\
अब, आप s32 का उपयोग **प्रशासक** के पूरे नाम के लिए हस्ताक्षर के रूप में कर सकते हैं।

### सारांश

1. उपयोगकर्ता नाम **Administ** (m1) का हस्ताक्षर प्राप्त करें जो s1 है
2. उपयोगकर्ता नाम **rator\x00\x00\x00 XOR s1 XOR 0** का हस्ताक्षर प्राप्त करें जो s32 है**.**
3. कुकी को s32 पर सेट करें और यह उपयोगकर्ता **प्रशासक** के लिए एक मान्य कुकी होगी।

# हमला नियंत्रण IV

यदि आप उपयोग किए गए IV को नियंत्रित कर सकते हैं तो हमला बहुत आसान हो सकता है।\
यदि कुकी केवल उपयोगकर्ता नाम एन्क्रिप्ट किया गया है, तो उपयोगकर्ता "**प्रशासक**" का उपयोगकर्ता बना सकते हैं और आप इसकी कुकी प्राप्त करेंगे।\
अब, यदि आप IV को नियंत्रित कर सकते हैं, तो आप IV का पहला बाइट बदल सकते हैं ताकि **IV\[0] XOR "A" == IV'\[0] XOR "a"** और उपयोगकर्ता **प्रशासक** के लिए कुकी पुनः उत्पन्न कर सकते हैं। यह कुकी **प्रशासक** के उपयोगकर्ता का अनुकरण करने के लिए मान्य होगी।

# संदर्भ

अधिक जानकारी [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** को PRs जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
