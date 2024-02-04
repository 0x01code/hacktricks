# SmbExec/ScExec

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PRs सबमिट करके।

</details>

## कैसे काम करता है

**Smbexec** **Psexec** की तरह काम करता है, जो विक्टिम के सिस्टम पर **cmd.exe** या **powershell.exe** को लक्ष्य बैकडोर कार्यान्वयन के लिए लक्षित करता है, जानकारीहीन एक्जीक्यूटेबल का उपयोग टालता है।

## **SMBExec**
```bash
smbexec.py WORKGROUP/username:password@10.10.10.10
```
Smbexec की क्षमता में शामिल है एक अस्थायी सेवा (उदाहरण के लिए, "BTOBTO") बनाना लक्षित मशीन पर बिनरी छोड़े बिना कमांड को निष्पादित करने के लिए। यह सेवा, जो कमांड को cmd.exe के पथ (%COMSPEC%) के माध्यम से चलाने के लिए निर्मित है, आउटपुट को एक अस्थायी फ़ाइल में पुनर्निर्देशित करती है और निष्पादन के बाद खुद को हटा देती है। यह विधि छिपकली है लेकिन प्रत्येक कमांड के लिए घटना लॉग उत्पन्न करती है, जो आक्रमणकर्ता की ओर से जारी किए गए प्रत्येक कमांड के लिए एक गैर-संवादात्मक "शैल" प्रदान करती है।

## बिनरी के बिना कमांड निष्पादन

इस दृष्टिकोण की अनुमति सीधे कमांड निष्पादन के लिए सेवा binPaths के माध्यम से, बिनरी की आवश्यकता को समाप्त करते हैं। यह खासकर विंडोज लक्ष्य पर एक-बार कमांड निष्पादन के लिए उपयुक्त है। उदाहरण के लिए, Metasploit के `web_delivery` मॉड्यूल का उपयोग करके PowerShell-लक्षित प्रतिलोम Meterpreter पेलोड के साथ एक सुनने वाला स्थापित कर सकते हैं जो आवश्यक निष्पादन कमांड प्रदान करता है। आक्रमणकर्ता की विंडोज मशीन पर एक दूरस्थ सेवा बनाना और शुरू करना, जिसमें binPath को इस कमांड को cmd.exe के माध्यम से निष्पादित करने के लिए सेट किया गया है, Metasploit सुनने वाले की ओर से कॉलबैक और पेलोड निष्पादन को प्राप्त करने के बावजूद संभावित सेवा प्रतिक्रिया त्रुटियों के बावजूद, प्राप्त करने की अनुमति देता है।

### कमांड उदाहरण

सेवा बनाने और शुरू करने के लिए निम्नलिखित कमांडों के साथ संभावित है:
```cmd
sc create [ServiceName] binPath= "cmd.exe /c [PayloadCommand]"
sc start [ServiceName]
```
# संदर्भ
* [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
