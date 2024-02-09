# Over Pass the Hash/Pass the Key

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस चाहिए**? [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो करें**.
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके**।

</details>

## Overpass The Hash/Pass The Key (PTK)

**Overpass The Hash/Pass The Key (PTK)** हमला उन environments के लिए डिज़ाइन किया गया है जहाँ पारंपरिक NTLM प्रोटोकॉल पर प्रतिबंध है, और Kerberos प्रमाणीकरण प्राथमिकता देता है। यह हमला एक उपयोगकर्ता के NTLM हैश या AES कुंजियों का उपयोग करता है ताकि Kerberos टिकट प्राप्त किया जा सके, जिससे नेटवर्क में संसाधनों तक के अनधिकृत पहुंच सके।

इस हमले को क्रियान्वित करने के लिए, पहला कदम लक्षित उपयोगकर्ता के खाते का NTLM हैश या पासवर्ड प्राप्त करना है। इस जानकारी को सुरक्षित करने के बाद, खाते के लिए टिकट ग्रांटिंग टिकट (TGT) प्राप्त किया जा सकता है, जिससे हमलावर को उन सेवाओं या मशीनों तक पहुंच मिल सकती है जिसकी अनुमति उपयोगकर्ता को है।

इस प्रक्रिया को निम्नलिखित कमांडों से प्रारंभ किया जा सकता है:
```bash
python getTGT.py jurassic.park/velociraptor -hashes :2a3de7fe356ee524cc9f3d579f2e0aa7
export KRB5CCNAME=/root/impacket-examples/velociraptor.ccache
python psexec.py jurassic.park/velociraptor@labwws02.jurassic.park -k -no-pass
```
उन परिस्थितियों के लिए जिनमें AES256 की आवश्यकता हो, `-aesKey [AES key]` विकल्प का उपयोग किया जा सकता है। इसके अतिरिक्त, प्राप्त टिकट का उपयोग कई उपकरणों के साथ किया जा सकता है, जैसे smbexec.py या wmiexec.py, हमले के दायरे को विस्तारित करते हुए।

_पाया गया समस्याएं_ जैसे _PyAsn1Error_ या _KDC cannot find the name_ आम तौर पर Impacket पुस्तकालय को अपडेट करके या IP पते की बजाय होस्टनाम का उपयोग करके हल किए जा सकते हैं, जिससे केरबेरोस KDC के साथ संगति सुनिश्चित हो।

Rubeus.exe का उपयोग करके एक वैकल्पिक कमांड क्रम इस तकनीक के एक अन्य पहलू को दर्शाता है:
```bash
.\Rubeus.exe asktgt /domain:jurassic.park /user:velociraptor /rc4:2a3de7fe356ee524cc9f3d579f2e0aa7 /ptt
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```
यह विधि **पास द हैश** दृष्टिकोण के साथ मिलती है, जिसमें ध्यान केंद्रित करना है और प्रमाणीकरण के उद्देश्यों के लिए प्रत्यक्ष रूप से टिकट का उपयोग करना। महत्वपूर्ण है कि एक टीजीटी अनुरोध की प्रारंभिकता घटक `4768: एक केरबेरोस प्रमाणीकरण टिकट (टीजीटी) का अनुरोध किया गया था` को सक्रिय करती है, जिससे डिफ़ॉल्ट रूप से RC4-HMAC का उपयोग किया जाता है, हालांकि आधुनिक Windows सिस्टम्स AES256 को अधिक पसंद करते हैं।

संचालन सुरक्षा को अनुरूप बनाने और AES256 का उपयोग करने के लिए निम्नलिखित कमांड लागू किया जा सकता है:
```bash
.\Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:HASH /nowrap /opsec
```
## संदर्भ

* [https://www.tarlogic.com/es/blog/como-atacar-kerberos/](https://www.tarlogic.com/es/blog/como-atacar-kerberos/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके**।

</details>
