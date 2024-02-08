# टोकनों का दुरुपयोग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन हैकट्रिक्स में देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करने का एक्सेस चाहिए**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** को** फॉलो करें।
* **अपने हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud)** में पीआर जमा करके।

</details>

## टोकन

अगर आपको **विंडोज एक्सेस टोकन्स क्या हैं नहीं पता** तो इस पेज को पढ़ें जारी रखें:

{% content-ref url="../access-tokens.md" %}
[access-tokens.md](../access-tokens.md)
{% endcontent-ref %}

**शायद आप उन टोकनों का दुरुपयोग करके विशेषाधिकारों को बढ़ा सकते हैं जो आपके पास पहले से हैं**

### SeImpersonatePrivilege

यह एक विशेषाधिकार है जो किसी प्रक्रिया द्वारा किसी भी टोकन का अनुकरण (लेकिन निर्माण नहीं) करने की अनुमति देता है, प्राप्त करने के लिए एक हैंडल प्राप्त किया जा सकता है। एक विशेषाधिकृत टोकन को एक विंडोज सेवा (DCOM) से प्राप्त किया जा सकता है जिसे NTLM प्रमाणीकरण के खिलाफ एक एक्सप्लॉइट के खिलाफ कार्रवाई करने के लिए प्रेरित किया जा सकता है, जिसके बाद SYSTEM विशेषाधिकारों के साथ प्रक्रिया का क्रियान्वयन किया जा सकता है। इस सुरक्षा गड़बड़ी का उपयोग कई उपकरणों का उपयोग करके किया जा सकता है, जैसे कि [juicy-potato](https://github.com/ohpe/juicy-potato), [RogueWinRM](https://github.com/antonioCoco/RogueWinRM) (जिसके लिए winrm को अक्षम करना आवश्यक है), [SweetPotato](https://github.com/CCob/SweetPotato), और [PrintSpoofer](https://github.com/itm4n/PrintSpoofer)।

{% content-ref url="../roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](../roguepotato-and-printspoofer.md)
{% endcontent-ref %}

{% content-ref url="../juicypotato.md" %}
[juicypotato.md](../juicypotato.md)
{% endcontent-ref %}

### SeAssignPrimaryPrivilege

यह **SeImpersonatePrivilege** के बहुत ही समान है, यह **एक ही विधि** का उपयोग करेगा एक विशेषाधिकृत टोकन प्राप्त करने के लिए।\
इसके बाद, यह विशेषाधिकार **नए/सस्पेंड किए गए प्रक्रिया को प्राथमिक टोकन सौंपने** की अनुमति देता है। विशेषाधिकृत अनुकरण टोकन के साथ आप एक प्राथमिक टोकन को उत्पन्न कर सकते हैं (DuplicateTokenEx)।\
टोकन के साथ, आप 'CreateProcessAsUser' के साथ **नई प्रक्रिया** बना सकते हैं या एक प्रक्रिया सस्पेंड कर सकते हैं और **टोकन सेट** कर सकते हैं (सामान्यत: एक चल रही प्रक्रिया के प्राथमिक टोकन को आप संशोधित नहीं कर सकते हैं)।

### SeTcbPrivilege

यदि आपने इस टोकन को सक्षम किया है तो आप **KERB\_S4U\_LOGON** का उपयोग करके किसी अन्य उपयोगकर्ता के लिए एक **अनुकरण टोकन** प्राप्त कर सकते हैं बिना पासवर्ड के जानकारी, **किसी भी विचित्र समूह** (व्यवस्थापक) को जोड़ सकते हैं, टोकन की **अखंडता स्तर** को "**मध्यम**" में सेट कर सकते हैं, और इस टोकन को **वर्तमान धागे** (SetThreadToken) को सौंप सकते हैं।

### SeBackupPrivilege

इस विशेषाधिकार द्वारा सभी पठन पहुंच नियंत्रण को किसी भी फ़ाइल को (केवल पठन कार्रवाइयों के लिए सीमित) प्रदान किया जाता है। इसका उपयोग स्थानीय प्रशासक के पासवर्ड हैश पठन के लिए किया जाता है, इसके बाद, "**psexec**" या "**wmicexec**" जैसे उपकरणों का उपयोग हैश के साथ किया जा सकता है (पास-द-हैश तकनीक)। हालांकि, यह तकनीक दो स्थितियों में विफल हो जाती है: जब स्थानीय प्रशासक खाता अक्षम होता है, या जब एक नीति लागू होती है जो स्थानीय प्रशासकों को दूरस्थ रूप से कनेक्ट करने से प्रशासनिक अधिकार हटा देती है।\
आप इस विशेषाधिकार का **दुरुपयोग** कर सकते हैं:

* [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)
* [https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)
* **IppSec** की [https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec](https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec) में अनुसरण करें
* या जैसा कि निम्नलिखित में स्पष्ट किया गया है **बैकअप ऑपरेटर्स के साथ विशेषाधिकारों को बढ़ाना** खंड में:

{% content-ref url="../../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### SeRestorePrivilege

इस विशेषाधिकार द्वारा किसी भी सिस्टम फ़ाइल के लिए **लेखन पहुंच** की अनुमति दी जाती है, चाहे फ़ाइल की पहुंच नियंत्रण सूची (ACL) हो या न हो। यह उचित रूप से उचित बनाता है अनेक उन्नतियों के लिए, जिसमें **सेवाओं को संशोधित करना**, DLL हाइजैकिंग करना, और विभिन्न तकनीकों में **डीबगर्स** सेट करना शामिल है।

### SeCreateTokenPrivilege

SeCreateTokenPrivilege एक शक्तिशाली अनुमति है, विशेष रूप से उपयुक्त जब एक उपयोगकर्ता के पास टोकन का अनुकरण करने की क्षमता होती है, लेकिन SeImpersonatePrivilege की अनुपस्थिति में भी। यह क्षमता उस समय निर्भर करती है जब विशिष्ट स्थितियों के तहत टोकनों का अनुकरण किया जा सकता है।
- **SeImpersonatePrivilege के बिना अनुकरण:** विशिष्ट स्थितियों के तहत टोकनों का अनुकरण करने के लिए SeCreateTokenPrivilege का उपयोग किया जा सकता है।
- **टोकन अनुकरण के लिए शर्तें:** सफल अनुकरण के लिए लक्षित टोकन का उस उप
```python
# Example Python code to set the registry values
import winreg as reg

# Define the path and values
path = r'Software\YourPath\System\CurrentControlSet\Services\DriverName' # Adjust 'YourPath' as needed
key = reg.OpenKey(reg.HKEY_CURRENT_USER, path, 0, reg.KEY_WRITE)
reg.SetValueEx(key, "ImagePath", 0, reg.REG_SZ, "path_to_binary")
reg.SetValueEx(key, "Type", 0, reg.REG_DWORD, 0x00000001)
reg.CloseKey(key)
```
इस विशेषाधिकार का दुरुपयोग करने के और तरीके [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege)

### SeTakeOwnershipPrivilege

यह **SeRestorePrivilege** के समान है। इसका मुख्य कार्य एक प्रक्रिया को **एक ऑब्जेक्ट के स्वामित्व का अनुमान** करने देता है, जिससे WRITE_OWNER पहुंच के आवश्यकता को दूर करता है। इस प्रक्रिया में पहले इच्छित रजिस्ट्री कुंजी के स्वामित्व को लिखने के उद्देश्य से सुरक्षित करना होता है, फिर DACL को लिखने के उद्देश्य से संशोधित करना होता है।
```bash
takeown /f 'C:\some\file.txt' #Now the file is owned by you
icacls 'C:\some\file.txt' /grant <your_username>:F #Now you have full access
# Use this with files that might contain credentials such as
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software
%WINDIR%\repair\security
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
c:\inetpub\wwwwroot\web.config
```
### SeDebugPrivilege

यह विशेषाधिकार **अन्य प्रक्रियाओं को डीबग करने** की अनुमति देता है, जिसमें मेमोरी में पढ़ने और लिखने की क्षमता शामिल है। इस विशेषाधिकार के साथ विभिन्न मेमोरी इंजेक्शन के लिए विभिन्न रणनीतियाँ लागू की जा सकती हैं, जो अधिकांश एंटीवायरस और होस्ट प्रवेश निवारण समाधानों को टाल सकती हैं।

#### मेमोरी डंप

आप [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) का उपयोग कर सकते हैं [SysInternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) से **किसी प्रक्रिया की मेमोरी को कैप्चर करने** के लिए। विशेष रूप से, यह **स्थानीय सुरक्षा प्राधिकरण उपस्थिति सेवा ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service))** प्रक्रिया पर लागू हो सकता है, जो एक उपयोगकर्ता ने सफलतापूर्वक सिस्टम में लॉगिन किया है उसके पासवर्ड संग्रहित करने के लिए जिम्मेदार है।

फिर आप इस डंप को मिमीकेट्ज में लोड कर सकते हैं ताकि पासवर्ड प्राप्त कर सकें:
```
mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```
#### RCE

यदि आप `NT SYSTEM` शैल प्राप्त करना चाहते हैं तो आप इस्तेमाल कर सकते हैं:

* ****[**SeDebugPrivilegePoC**](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC)****
* ****[**psgetsys.ps1**](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1)****
```powershell
# Get the PID of a process running as NT SYSTEM
import-module psgetsys.ps1; [MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>)
```
## विशेषाधिकारों की जाँच
```
whoami /priv
```
**टोकन जो अक्षम दिखाई देते हैं** को सक्रिय किया जा सकता है, आप _सक्रिय_ और _अक्षम_ टोकन का दुरुपयोग कर सकते हैं।

### सभी टोकन सक्रिय करें

यदि आपके पास अक्षम टोकन हैं, तो आप [**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) स्क्रिप्ट का उपयोग करके सभी टोकन को सक्रिय कर सकते हैं:
```powershell
.\EnableAllTokenPrivs.ps1
whoami /priv
```
या **स्क्रिप्ट** इस [**पोस्ट**](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/) में एम्बेड करें।

## तालिका

पूर्ण टोकन विशेषाधिकार चीटशीट पर [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin), सारांश नीचे केवल प्रिविलेज को उत्पीड़ित करने के सीधे तरीके सूचीबद्ध करेगा जिससे एडमिन सत्र प्राप्त किया जा सकता है या संवेदनशील फ़ाइलें पढ़ी जा सकती हैं।

| विशेषाधिकार              | प्रभाव       | उपकरण                   | निष्पादन पथ                                                                                                                                                                                                                                                                                                                                     | टिप्पणियाँ                                                                                                                                                                                                                                                                                                                        |
| -------------------------- | ----------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`SeAssignPrimaryToken`** | _**एडमिन**_ | तृतीय पक्ष उपकरण          | _"यह एक उपयोगकर्ता को टोकन का अनुकरण करने और उपकरणों का उपयोग करके nt सिस्टम में privesc करने की अनुमति देगा जैसे कि potato.exe, rottenpotato.exe और juicypotato.exe"_                                                                                                                                                                                                      | अपडेट के लिए आपका धन्यवाद [Aurélien Chalot](https://twitter.com/Defte\_)। मैं जल्द ही इसे कुछ और रेसिपी जैसा बनाने की कोशिश करूंगा।                                                                                                                                                                                        |
| **`SeBackup`**             | **खतरा**   | _**अंतर्निहित कमांड**_ | `robocopy /b` के साथ संवेदनशील फ़ाइलें पढ़ें                                                                                                                                                                                                                                                                                                             | <p>- यदि आप %WINDIR%\MEMORY.DMP पढ़ सकते हैं तो यह और भी दिलचस्प हो सकता है।<br><br>- <code>SeBackupPrivilege</code> (और robocopy) खुली फ़ाइलों के मामले में मददगार नहीं है।<br><br>- Robocopy को /b पैरामीटर के साथ काम करने के लिए दोनों SeBackup और SeRestore की आवश्यकता है।</p>                                                                      |
| **`SeCreateToken`**        | _**एडमिन**_ | तृतीय पक्ष उपकरण          | `NtCreateToken` के साथ स्थानिक एडमिन अधिकारों सहित विविध टोकन बनाएं।                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                                                |
| **`SeDebug`**              | _**एडमिन**_ | **PowerShell**          | `lsass.exe` टोकन की नकल करें।                                                                                                                                                                                                                                                                                                                   | [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1) पर स्क्रिप्ट देखें।                                                                                                                                                                                                         |
| **`SeLoadDriver`**         | _**एडमिन**_ | तृतीय पक्ष उपकरण          | <p>1. <code>szkg64.sys</code> जैसे बगी कर्नेल ड्राइवर लोड करें<br>2. ड्राइवर की कमजोरी का शोषण करें<br><br>वैकल्पिक रूप से, यह विशेषाधिकार <code>ftlMC</code> इनबिल्ट कमांड के साथ सुरक्षा संबंधित ड्राइवरों को अनलोड करने के लिए उपयोग किया जा सकता है। जैसे: <code>fltMC sysmondrv</code></p>                                                                           | <p>1. <code>szkg64</code> की कमजोरी <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732">CVE-2018-15732</a> के रूप में सूचीबद्ध है<br>2. <code>szkg64</code> <a href="https://www.greyhathacker.net/?p=1025">एक्सप्लॉइट कोड</a> को <a href="https://twitter.com/parvezghh">Parvez Anwar</a> ने बनाया था</p> |
| **`SeRestore`**            | _**एडमिन**_ | **PowerShell**          | <p>1. SeRestore विशेषाधिकार के साथ PowerShell/ISE लॉन्च करें।<br>2. <a href="https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1">Enable-SeRestorePrivilege</a> के साथ विशेषाधिकार सक्षम करें।<br>3. utilman.exe को utilman.old में नाम बदलें<br>4. cmd.exe को utilman.exe में नाम बदलें<br>5. कंसोल लॉक करें और Win+U दबाएं</p> | <p>कुछ AV सॉफ़्टवेयर द्वारा हमला पहचाना जा सकता है।</p><p>वैकल्पिक विधि सेवा बाइनरी को एक ही विशेषाधिकार का उपयोग करके "प्रोग्राम फ़ाइल्स" में स्टोर किए गए बाइनरी को बदलने पर निर्भर करती है।</p>                                                                                                                                                            |
| **`SeTakeOwnership`**      | _**एडमिन**_ | _**अंतर्निहित कमांड**_ | <p>1. <code>takeown.exe /f "%windir%\system32"</code><br>2. <code>icalcs.exe "%windir%\system32" /grant "%username%":F</code><br>3. cmd.exe को utilman.exe में नाम बदलें<br>4. कंसोल लॉक करें और Win+U दबाएं</p>                                                                                                                                       | <p>कुछ AV सॉफ़्टवेयर द्वारा हमला पहचाना जा सकता है।</p><p>वैकल्पिक विधि सेवा बाइनरी को एक ही विशेषाधिकार का उपयोग करके "प्रोग्राम फ़ाइल्स" में स्टोर किए गए बाइनरी को बदलने पर निर्भर करती है।</p>                                                                                                                                                           |
| **`SeTcb`**                | _**एडमिन**_ | तृतीय पक्ष उपकरण          | <p>लोकल एडमिन अधिकारों को शामिल करने के लिए टोकन को मानिपुरेट करें। SeImpersonate की आवश्यकता हो सकती है।</p><p>सत्यापित करने के लिए।</p>                                                                                                                                                                                                                                     |                                                                                                                                                                                                                                                                                                                                |

## संदर्भ

* विंडोज टोकन को परिभाषित करने वाली इस तालिका को देखें: [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)
* टोकन के साथ privesc के बारे में [**इस पेपर**](https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt) को देखें।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण को डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स रेपो (https://github.com/carlospolop/hacktricks) और हैकट्रिक्स-क्लाउड रेपो (https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।

</details>
