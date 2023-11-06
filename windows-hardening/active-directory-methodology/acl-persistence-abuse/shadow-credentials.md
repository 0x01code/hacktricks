# छाया क्रेडेंशियल्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके अपना योगदान दें।**

</details>

## परिचय <a href="#3f17" id="3f17"></a>

इस तकनीक के बारे में [**सभी जानकारी के लिए मूल पोस्ट की जांच करें**](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)।

संक्षेप में: यदि आप किसी उपयोगकर्ता/कंप्यूटर के **msDS-KeyCredentialLink** गुण के लिए लिख सकते हैं, तो आप उस ऑब्जेक्ट के **NT हैश** को प्राप्त कर सकते हैं।

इसलिए आप ऑब्जेक्ट के लिए **सार्वजनिक-निजी कुंजी प्रमाणीकरण क्रेडेंशियल्स** सेट कर सकेंगे और उन्हें उपयोग करके एक **विशेष सेवा टिकट** प्राप्त कर सकेंगे जिसमें एनटीएलएम हैश एन्क्रिप्टेड NTLM\_SUPPLEMENTAL\_CREDENTIAL एंटिटी के भीतर निहित होता है जिसे आप डिक्रिप्ट कर सकते हैं।

### आवश्यकताएं <a href="#2de4" id="2de4"></a>

इस तकनीक के लिए निम्नलिखित आवश्यकताएं होती हैं:

* कम से कम एक Windows Server 2016 डोमेन कंट्रोलर।
* डोमेन कंट्रोलर पर सर्वर प्रमाणीकरण के लिए एक डिजिटल प्रमाणपत्र।
* Active Directory में Windows Server 2016 कार्यान्वयन स्तर।
* लक्षित ऑब्जेक्ट के msDS-KeyCredentialLink गुण में लिखने के लिए धारित अधिकार वाले खाते को संक्रमित करें।

## दुरुपयोग

कंप्यूटर ऑब्जेक्ट के लिए Key Trust का दुरुपयोग करने के लिए, खाते के लिए एक TGT और NTLM हैश प्राप्त करने के बाद अतिरिक्त कदम चाहिए होते हैं। सामान्यतः दो विकल्प होते हैं:

1. एक **RC4 सिल्वर टिकट** जालसाजी करें ताकि उच्चाधिकृत उपयोगकर्ताओं को संबंधित होस्ट के लिए अनुकरण कर सकें।
2. TGT का उपयोग करके **S4U2Self** को कॉल करें ताकि उच्चाधिकृत उपयोगकर्ताओं को संबंधित होस्ट के लिए अनुकरण कर सकें। इस विकल्प के लिए, प्राप्त सेवा टिकट में सेवा नाम में एक सेवा वर्ग शामिल करने की आवश्यकता होती है।

Key Trust दुरुपयोग का अतिरिक्त लाभ है कि इसमें एक और खाता को पहुंच दी जाती है जिसे संक्रमित किया जा सकता है - यह **हमारे द्वारा उत्पन्न की गई आकस्मिक कुंजी सीमित होती है**। इसके अलावा, यह एक कंप्यूटर खाता बनाने की आवश्यकता नहीं है जिसे उच्चाध
## [pywhisker](https://github.com/ShutdownRepo/pywhisker) <a href="#7e2e" id="7e2e"></a>

pyWhisker एक Python संस्करण है जो Elad Shamir द्वारा बनाए गए मूल Whisker का समकक्ष है और C# में लिखा गया है। यह उपकरण उपयोगकर्ताओं को लक्ष्य उपयोगकर्ता / कंप्यूटर के msDS-KeyCredentialLink गुणांक को संशोधित करने की अनुमति देता है ताकि उन्हें उस वस्तु पर पूर्ण नियंत्रण मिल सके।

यह Impacket पर आधारित है और podalirius द्वारा बनाए गए Michael Grafnetter के DSInternals का Python संकक्ष है। यह उपकरण, Dirk-jan के PKINITtools के साथ केवल UNIX आधारित सिस्टमों पर पूर्ण प्राथमिक उत्पीड़न की अनुमति देता है।


pyWhisker का उपयोग निम्नलिखित क्रियाओं को msDs-KeyCredentialLink गुणांक पर करने के लिए किया जा सकता है

- *सूची*: सभी मौजूदा KeyCredentials ID और निर्माण समय की सूची बनाएँ
- *जानकारी*: KeyCredential संरचना में सभी जानकारी प्रिंट करें
- *जोड़ें*: msDs-KeyCredentialLink में एक नया KeyCredential जोड़ें
- *हटाएं*: msDs-KeyCredentialLink से एक KeyCredential हटाएं
- *साफ़ करें*: msDs-KeyCredentialLink से सभी KeyCredentials हटाएं
- *निर्यात*: JSON में msDs-KeyCredentialLink से सभी KeyCredentials निर्यात करें
- *आयात*: JSON फ़ाइल से KeyCredentials के साथ msDs-KeyCredentialLink को अधिलेखित करें


pyWhisker निम्नलिखित प्रमाणीकरण का समर्थन करता है:
- (NTLM) साफ पाठशब्द
- (NTLM) पास-द-हैश
- (Kerberos) साफ पाठशब्द
- (Kerberos) पास-द-कुंजी / ओवरपास-द-हैश
- (Kerberos) पास-द-कैश (पास-द-टिकट के प्रकार)

![](https://github.com/ShutdownRepo/pywhisker/blob/main/.assets/add_pfx.png)


{% hint style="info" %}
[**Readme**](https://github.com/ShutdownRepo/pywhisker) पर अधिक विकल्प।
{% endhint %}

## [ShadowSpray](https://github.com/Dec0ne/ShadowSpray/)

कई मामलों में, समूह "Everyone" / "Authenticated Users" / "Domain Users" या कोई अन्य **व्यापक समूह** डोमेन में सभी उपयोगकर्ताओं को सम्प्रदाय में कुछ `GenericWrite`/`GenericAll` DACLs **अन्य वस्तुओं** पर होते हैं। [**ShadowSpray**](https://github.com/Dec0ne/ShadowSpray/) इसलिए उन सभी पर **ShadowCredentials** का दुरुपयोग करने का प्रयास करता है

यह कुछ इस तरह से होता है:

1. प्रदान की गई प्रमाणिकता के साथ डोमेन में **लॉगिन** करें (या मौजूदा सत्र का उपयोग करें)।
2. यह जांचें कि **डोमेन कार्यात्मक स्तर 2016 है** (अन्यथा रुकें क्योंकि Shadow Credentials हमला काम नहीं करेगा)
3. LDAP से डोमेन में सभी वस्तुओं की **सूची इकट्ठा करें** (उपयोगकर्ता और कंप्यूटर)
4. सूची में **प्रत्येक वस्तु** के लिए निम्नलिखित करें:
1. वस्तु के `msDS-KeyCredentialLink` गुणांक में **KeyCredential जोड़ने** का प्रयास करें।
2. यदि उपरोक्त **सफल है**, तो जोड़े गए KeyCredential का उपयोग करके **PKINIT** का अनुरोध करें और जोड़ा गया KeyCredential का उपयोग करके **TGT** अनुरोध करें।
3. यदि उपरोक्त **सफल है**, तो **UnPACTheHash** हमला करें और उपयोगकर्ता / कंप्यूटर **NT हैश** उजागर करें।
4. यदि **`--RestoreShadowCred`** निर्दिष्ट किया गया है: जोड़े गए KeyCredential को हटाएं (अपने आप को साफ करें...)
5. यदि **`--Recursive`** निर्दिष्ट किया गया है: हमारे पास सफलतापूर्वक स्वामित्व में होने वाले प्रत्येक उपयोगकर्ता / कंप्यूटर खाते का उपयोग करके **एक ही प्रक्रिया** करें।

## संदर्भ

* [https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)
* [https://github.com/eladshamir/Whisker](https://github.com/eladshamir/Whisker)
* [https://github.com/Dec0ne/ShadowSpray/](https://github.com/Dec0ne/ShadowSpray/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की अनुमति चाहिए? [**सदस्यता योज
