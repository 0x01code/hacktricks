<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>


# CBC

यदि **cookie** **केवल** **username** है (या cookie का पहला भाग username है) और आप "**admin**" username का अनुकरण करना चाहते हैं। तब, आप username **"bdmin"** बना सकते हैं और cookie के **पहले बाइट** का **bruteforce** कर सकते हैं।

# CBC-MAC

क्रिप्टोग्राफी में, **cipher block chaining message authentication code** (**CBC-MAC**) एक ब्लॉक सिफर से मैसेज ऑथेंटिकेशन कोड बनाने की तकनीक है। मैसेज को किसी ब्लॉक सिफर एल्गोरिथम में CBC मोड में एन्क्रिप्ट किया जाता है ताकि **ब्लॉक्स की एक श्रृंखला बने जिसमें प्रत्येक ब्लॉक पिछले ब्लॉक के सही एन्क्रिप्शन पर निर्भर करता है**। यह आपसी निर्भरता सुनिश्चित करती है कि प्लेनटेक्स्ट के **किसी भी** **बिट्स** में **परिवर्तन** से **अंतिम एन्क्रिप्टेड ब्लॉक** में ऐसा **परिवर्तन** होगा जिसे ब्लॉक सिफर की कुंजी जाने बिना पूर्वानुमानित या प्रतिकार नहीं किया जा सकता।

मैसेज m का CBC-MAC की गणना करने के लिए, m को CBC मोड में शून्य प्रारंभिक वेक्टर के साथ एन्क्रिप्ट किया जाता है और अंतिम ब्लॉक को रखा जाता है। निम्नलिखित चित्र गुप्त कुंजी k और ब्लॉक सिफर E का उपयोग करके ब्लॉक्स![m\_{1}\\|m\_{2}\\|\cdots \\|m\_{x}](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) की एक मैसेज का CBC-MAC की गणना करता है:

![CBC-MAC structure (en).svg](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnerability

CBC-MAC में आमतौर पर **प्रयुक्त IV 0 होता है**।\
यह एक समस्या है क्योंकि 2 ज्ञात मैसेज (`m1` और `m2`) स्वतंत्र रूप से 2 हस्ताक्षर (`s1` और `s2`) उत्पन्न करेंगे। इसलिए:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

तब m1 और m2 को जोड़कर बनाए गए मैसेज (m3) से 2 हस्ताक्षर (s31 और s32) उत्पन्न होंगे:

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**जिसे एन्क्रिप्शन की कुंजी जाने बिना गणना की जा सकती है।**

कल्पना कीजिए कि आप **8bytes** ब्लॉक्स में नाम **Administrator** को एन्क्रिप्ट कर रहे हैं:

* `Administ`
* `rator\00\00\00`

आप **Administ** (m1) नाम का एक username बना सकते हैं और हस्ताक्षर (s1) प्राप्त कर सकते हैं।\
फिर, आप `rator\00\00\00 XOR s1` के परिणाम का एक username बना सकते हैं। यह `E(m2 XOR s1 XOR 0)` उत्पन्न करेगा जो s32 है।\
अब, आप s32 का उपयोग पूरे नाम **Administrator** के हस्ताक्षर के रूप में कर सकते हैं।

### सारांश

1. username **Administ** (m1) का हस्ताक्षर प्राप्त करें जो s1 है
2. username **rator\x00\x00\x00 XOR s1 XOR 0** का हस्ताक्षर s32 है।
3. cookie को s32 पर सेट करें और यह user **Administrator** के लिए एक वैध cookie होगी।

# IV को नियंत्रित करके हमला

यदि आप प्रयुक्त IV को नियंत्रित कर सकते हैं तो हमला बहुत आसान हो सकता है।\
यदि cookies केवल username को एन्क्रिप्ट किया गया है, तो user "**administrator**" का अनुकरण करने के लिए आप user "**Administrator**" बना सकते हैं और आपको इसकी cookie मिल जाएगी।\
अब, यदि आप IV को नियंत्रित कर सकते हैं, तो आप IV के पहले बाइट को बदल सकते हैं ताकि **IV\[0] XOR "A" == IV'\[0] XOR "a"** हो और user **Administrator** के लिए cookie को फिर से उत्पन्न करें। यह cookie user **administrator** का अनुकरण करने के लिए प्रारंभिक **IV** के साथ वैध होगी।

# संदर्भ

अधिक जानकारी के लिए [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC) पर जाएं।


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
