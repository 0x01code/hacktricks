# टोकनों का दुरुपयोग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस** चाहिए? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**दी पीएएस परिवार**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएएस और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) **डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) **में या** मुझे **ट्विटर** पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में।**

</details>

## टोकन

अगर आपको **Windows एक्सेस टोकन्स क्या हैं नहीं पता** तो इस पेज को पढ़ें:

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

**शायद आप उन टोकनों का दुरुपयोग करके विशेषाधिकारों को बढ़ा सकते हैं जो आपके पास पहले से हैं**

### SeImpersonatePrivilege

यह एक विशेषाधिकार है जो किसी प्रक्रिया द्वारा किसी भी टोकन का अनुकरण (लेकिन निर्माण नहीं) करने की अनुमति देता है, प्राप्त किया जा सकता है कि उसका हैंडल प्राप्त किया जा सकता है। एक विशेषाधिकृत टोकन को विंडोज सेवा (DCOM) से प्राप्त किया जा सकता है जिसे NTLM प्रमाणीकरण का उपयोग करके एक एक्सप्लॉइट के खिलाफ कार्रवाई करने के लिए प्रेरित किया जा सकता है, जिसके बाद SYSTEM विशेषाधिकारों के साथ प्रक्रिया का क्रियान्वयन संभव हो जाता है। इस सुरक्षा गड़बड़ी का उपयोग करने के लिए विभिन्न उपकरणों का उपयोग किया जा सकता है, जैसे कि [juicy-potato](https://github.com/ohpe/juicy-potato), [RogueWinRM](https://github.com/antonioCoco/RogueWinRM) (जिसके लिए winrm को अक्षम करना आवश्यक है), [SweetPotato](https://github.com/CCob/SweetPotato), और [PrintSpoofer](https://github.com/itm4n/PrintSpoofer)।

{% content-ref url="roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](roguepotato-and-printspoofer.md)
{% endcontent-ref %}

{% content-ref url="juicypotato.md" %}
[juicypotato.md](juicypotato.md)
{% endcontent-ref %}

### SeAssignPrimaryPrivilege

यह **SeImpersonatePrivilege** के बहुत ही समान है, यह **उसी विधि** का उपयोग करेगा एक विशेषाधिकृत टोकन प्राप्त करने के लिए।\
इसके बाद, यह विशेषाधिकार **नए/सस्पेंड किए गए प्रक्रिया को प्राथमिक टोकन को सौंपने** की अनुमति देता है। विशेषाधिकृत अनुकरण टोकन के साथ आप एक प्राथमिक टोकन को उत्पन्न कर सकते हैं (DuplicateTokenEx)।\
इस टोकन के साथ, आप 'CreateProcessAsUser' के साथ एक **नई प्रक्रिया** बना सकते हैं या एक प्रक्रिया सस्पेंड कर सकते हैं और **टोकन सेट** कर सकते हैं (सामान्यत: चल रही प्रक्रिया का प्राथमिक टोकन संशोधित नहीं किया जा सकता है)।

### SeTcbPrivilege

यदि आपने इस टोकन को सक्षम किया है तो आप **KERB\_S4U\_LOGON** का उपयोग करके किसी अन्य उपयोगकर्ता के लिए एक **अनुकरण टोकन** प्राप्त कर सकते हैं बिना पासवर्ड जाने, **किसी भी विचित्र समूह** (व्यवस्थापक) को जोड़ सकते हैं, टोकन की **अखंडता स्तर** को "**मध्यम**" में सेट कर सकते हैं, और इस टोकन को **वर्तमान धागे** (SetThreadToken) को सौंप सकते हैं।

### SeBackupPrivilege

इस विशेषाधिकार द्वारा सिस्टम को किसी भी फ़ाइल को सभी पढ़ने की अनुमति (केवल पढ़ने के कार्रवाई के लिए सीमित) दी जाती है। यह **स्थानीय प्रशासक** खातों के पासवर्ड हैश पढ़ने के लिए उपयोग किया जाता है, इसके बाद, "**psexec**" या "**wmicexec**" जैसे उपकरणों का उपयोग हैश के साथ किया जा सकता है (पास-द-हैश तकनीक)। हालांकि, यह तकनीक दो स्थितियों में विफल हो जाती है: जब स्थानीय प्रशासक खाता अक्षम होता है, या जब एक नीति लागू होती है जो स्थानीय प्रशासकों से दूरस्थ जुड़ने से प्रशासनिक अधिकारों को हटा देती है।\
आप इस विशेषाधिकार का **दुरुपयोग** कर सकते हैं:

* [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)
* [https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)
* **IppSec** का पालन करें [https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec](https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec)
* या जैसा कि **बैकअप ऑपरेटर्स के साथ विशेषाधिकारों को बढ़ाने** के खंड में स्पष्ट किया गया है:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### SeRestorePrivilege

इस विशेषाधिकार द्वारा किसी भी सिस्टम फ़ाइल के लिए **लेखन अधिकार** प्रदान किए जाते हैं, चाहे फ़ाइल की पहुंच नियंत्रण सूची (ACL) हो या न हो। इससे उचित उन्नति के कई संभावनाएँ खुलती हैं, जैसे कि **सेवाओं को संशोधित** करना, DLL हाइजैकिंग करना, और इमेज फ़ाइल निष्पादन विकल्प के माध्यम से **डीबगर्स सेट** करना इत्यादि।

### SeCreateTokenPrivilege

SeCreateTokenPrivilege एक शक्तिशाली अनुमति है, विशेष रूप से उपयोक्ता के पास टोकन का अनुकरण करने की क्षमता होने पर उपयोगी है, लेकिन SeImpersonatePrivilege की अनुपस्थिति में भी। यह क्षमता उस उपयोक्ता का प्रतिनिधित्व करने वाले टोकन का अनुकरण करने की क्षमता पर निर्भर करती है जो वर्तमान प्रक्रिया की अखंडता स्तर से अधिक नहीं है।

**मुख्य बिंदु:**

* **SeImpersonatePrivilege के बिना अनुकरण:** विशेष परिस्थितियों के तहत टोकनों का अनुकरण करने के लिए SeCreateTokenPrivilege का उपयोग किया जा सकता है ईओपी के लिए।
* **टोकन अनुकरण के लिए शर्तें:** सफल अनुकरण के लिए लक्षित टोकन का उपयोगकर्ता के लिए होना चाहिए और उसका अखंडता स्तर उस प्रक्रिया के अ
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

यह **SeRestorePrivilege** के समान है। इसका मुख्य कार्यक्षेत्र एक प्रक्रिया को **एक ऑब्जेक्ट के स्वामित्व को अस्सुम करने** की अनुमति देना है, WRITE\_OWNER पहुँच के अधिकार के माध्यम से विवेकाधीन पहुँच की आवश्यकता को दूर करते हुए। इस प्रक्रिया में पहले इच्छित रजिस्ट्री कुंजी के स्वामित्व को लेखन के उद्देश्य से सुरक्षित करना होता है, फिर DACL को लिखने के लिए कार्यवाही करनी होती है।
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

यह प्रिविलेज **अन्य प्रक्रियाओं को डिबग करने** की अनुमति देता है, जिसमें मेमोरी में पढ़ने और लिखने की भी शामिल है। इस प्रिविलेज के साथ अधिकांश एंटीवायरस और होस्ट इन्ट्रूजन प्रिवेंशन समाधानों को टालने की क्षमता वाले विभिन्न मेमोरी इंजेक्शन के लिए विभिन्न रणनीतियाँ लागू की जा सकती हैं।

#### मेमोरी डंप

आप [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) का उपयोग कर सकते हैं [SysInternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) से **किसी प्रक्रिया की मेमोरी को** **कैप्चर करने** के लिए। विशेष रूप से, यह **स्थानीय सुरक्षा प्राधिकरण उपस्थिति सेवा (**[**LSASS**](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service)**)** प्रक्रिया पर लागू हो सकता है, जो एक उपयोगकर्ता ने सफलतापूर्वक सिस्टम में लॉगिन किया होने पर उपयोगकर्ता क्रेडेंशियल्स को संग्रहित करने के लिए जिम्मेदार है।

फिर आप इस डंप को मिमीकेट्ज में लोड कर सकते हैं ताकि पासवर्ड प्राप्त कर सकें:
```
mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```
#### RCE

यदि आप `NT SYSTEM` शैल (shell) प्राप्त करना चाहते हैं तो आप निम्नलिखित का उपयोग कर सकते हैं:

* [**SeDebugPrivilege-Exploit (C++)**](https://github.com/bruno-1337/SeDebugPrivilege-Exploit)
* [**SeDebugPrivilegePoC (C#)**](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC)
* [**psgetsys.ps1 (Powershell Script)**](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1)
```powershell
# Get the PID of a process running as NT SYSTEM
import-module psgetsys.ps1; [MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>)
```
## विशेषाधिकारों की जाँच
```
whoami /priv
```
**टोकन जो अक्षम रूप से प्रकट होते हैं** को सक्षम किया जा सकता है, आप _सक्षम_ और _अक्षम_ टोकन का दुरुपयोग कर सकते हैं।

### सभी टोकनों को सक्षम करें

यदि आपके पास अक्षम टोकन हैं, तो आप [**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) स्क्रिप्ट का उपयोग करके सभी टोकनों को सक्षम कर सकते हैं:
```powershell
.\EnableAllTokenPrivs.ps1
whoami /priv
```
या **स्क्रिप्ट** इस [**पोस्ट**](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/) में एम्बेड करें।

## तालिका

पूर्ण टोकन विशेषाधिकार चीटशीट पर [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin), सारांश नीचे केवल विशेषाधिकार का शोषण करने के सीधे तरीके सूचीबद्ध करेगा जिससे एडमिन सत्र प्राप्त किया जा सके या संवेदनशील फ़ाइलें पढ़ी जा सकें।

| विशेषाधिकार              | प्रभाव      | उपकरण                  | निष्पादन पथ                                                                                                                                                                                                                                                                                                                                     | टिप्पणियाँ                                                                                                                                                                                                                                                                                                                        |
| -------------------------- | ----------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`SeAssignPrimaryToken`** | _**एडमिन**_ | तृतीय पक्षीय उपकरण          | _"यह एक उपयोगकर्ता को टोकन का अनुकरण करने और उपकरणों का उपयोग करके nt सिस्टम में privesc करने की अनुमति देगा जैसे कि potato.exe, rottenpotato.exe और juicypotato.exe"_                                                                                                                                                                                                      | अपडेट के लिए धन्यवाद [Aurélien Chalot](https://twitter.com/Defte\_)। मैं जल्द ही इसे कुछ और रेसिपी जैसा बनाने की कोशिश करूंगा।                                                                                                                                                                                        |
| **`SeBackup`**             | **खतरा**   | _**बिल्ट-इन कमांड्स**_ | `robocopy /b` के साथ संवेदनशील फ़ाइलें पढ़ें                                                                                                                                                                                                                                                                                                             | <p>- यदि आप %WINDIR%\MEMORY.DMP पढ़ सकते हैं तो यह और भी रोचक हो सकता है<br><br>- <code>SeBackupPrivilege</code> (और robocopy) फ़ाइलें खोलने के समय मददगार नहीं हैं।<br><br>- Robocopy को /b पैरामीटर के साथ काम करने के लिए दोनों SeBackup और SeRestore की आवश्यकता है।</p>                                                                      |
| **`SeCreateToken`**        | _**एडमिन**_ | तृतीय पक्षीय उपकरण          | `NtCreateToken` के साथ स्थानिक एडमिन अधिकारों सहित विविध टोकन बनाएं।                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                                                |
| **`SeDebug`**              | _**एडमिन**_ | **PowerShell**          | `lsass.exe` टोकन की नकल करें।                                                                                                                                                                                                                                                                                                                   | [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1) पर स्क्रिप्ट खोजें                                                                                                                                                                                                         |
| **`SeLoadDriver`**         | _**एडमिन**_ | तृतीय पक्षीय उपकरण          | <p>1. <code>szkg64.sys</code> जैसे बगी कर्नेल ड्राइवर लोड करें<br>2. ड्राइवर की कमजोरी का शोषण करें<br><br>वैकल्पिक रूप से, यह विशेषाधिकार <code>ftlMC</code> बिल्ट-इन कमांड का उपयोग करके सुरक्षा संबंधित ड्राइवरों को अनलोड करने के लिए उपयोग किया जा सकता है। जैसे: <code>fltMC sysmondrv</code></p>                                                                           | <p>1. <code>szkg64</code> कमजोरी को <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732">CVE-2018-15732</a> के रूप में सूचीबद्ध किया गया है<br>2. <code>szkg64</code> <a href="https://www.greyhathacker.net/?p=1025">एक्सप्लॉइट कोड</a> को <a href="https://twitter.com/parvezghh">Parvez Anwar</a> ने बनाया था</p> |
| **`SeRestore`**            | _**एडमिन**_ | **PowerShell**          | <p>1. SeRestore विशेषाधिकार के साथ PowerShell/ISE लॉन्च करें।<br>2. <a href="https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1">Enable-SeRestorePrivilege</a> के साथ विशेषाधिकार सक्षम करें।<br>3. utilman.exe को utilman.old में नाम बदलें<br>4. cmd.exe को utilman.exe में नाम बदलें<br>5. कंसोल लॉक करें और Win+U दबाएं</p> | <p>कुछ AV सॉफ़्टवेयर द्वारा हमला पहचाना जा सकता है।</p><p>वैकल्पिक विधि सेवा बाइनरी को एक ही विशेषाधिकार का उपयोग करके "प्रोग्राम फ़ाइल्स" में स्टोर किए गए बाइनरी को बदलने पर निर्भर करती है</p>                                                                                                                                                            |
| **`SeTakeOwnership`**      | _**एडमिन**_ | _**बिल्ट-इन कमांड्स**_ | <p>1. <code>takeown.exe /f "%windir%\system32"</code><br>2. <code>icalcs.exe "%windir%\system32" /grant "%username%":F</code><br>3. cmd.exe को utilman.exe में नाम बदलें<br>4. कंसोल लॉक करें और Win+U दबाएं</p>                                                                                                                                       | <p>कुछ AV सॉफ़्टवेयर द्वारा हमला पहचाना जा सकता है।</p><p>वैकल्पिक विधि सेवा बाइनरी को एक ही विशेषाधिकार का उपयोग करके "प्रोग्राम फ़ाइल्स" में स्टोर किए गए बाइनरी को बदलने पर निर्भर करती है।</p>                                                                                                                                                           |
| **`SeTcb`**                | _**एडमिन**_ | तृतीय पक्षीय उपकरण          | <p>लोकल एडमिन अधिकारों को शामिल करने के लिए टोकन को मानिपुलेट करें। SeImpersonate की आवश्यकता हो सकती है।</p><p>सत्यापित करने के लिए।</p>                                                                                                                                                                                                                                     |                                                                                                                                                                                                                                                                                                                                |

## संदर्भ

* विंडोज टोकनों को परिभाषित करने वाली इस तालिका को देखें: [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)
* [**इस पेपर**](https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt) को टोकन के साथ privesc के बारे में देखें।

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.** का **अनुसरण** करें।
* **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स रेपो (https://github.com/carlospolop/hacktricks) और हैकट्रिक्स-क्लाउड रेपो (https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।

</details>
