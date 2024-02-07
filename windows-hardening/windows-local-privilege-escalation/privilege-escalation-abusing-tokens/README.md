# टोकनों का दुरुपयोग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **ट्विटर** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**।
* **अपने हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>

## टोकन

अगर आपको **विंडोज एक्सेस टोकन्स क्या हैं** नहीं पता है तो इस पेज को पढ़ें:

{% content-ref url="../access-tokens.md" %}
[access-tokens.md](../access-tokens.md)
{% endcontent-ref %}

**शायद आप उन टोकनों का दुरुपयोग करके विशेषाधिकारों को बढ़ा सकते हैं जो आपके पास पहले से हैं**

### SeImpersonatePrivilege (3.1.1)

इस विशेषाधिकार को धारण करने वाला कोई भी प्रक्रिया **अनुकरण** कर सकता है (लेकिन नया नहीं बना सकता) किसी भी **टोकन** के लिए जिसको वह हैंडल प्राप्त कर सकता है। आप एक **विशेषाधिकृत टोकन** प्राप्त कर सकते हैं एक **विंडोज सेवा** (DCOM) से (एक्सप्लॉइट के खिलाफ एक **एनटीएलएम प्रमाणीकरण** कराकर), फिर **SYSTEM** के रूप में प्रक्रिया चला सकते हैं। [juicy-potato](https://github.com/ohpe/juicy-potato), [RogueWinRM ](https://github.com/antonioCoco/RogueWinRM)(winrm अक्षम होना चाहिए), [SweetPotato](https://github.com/CCob/SweetPotato), [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) के साथ इसका एक्सप्लॉइटेशन करें:

{% content-ref url="../roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](../roguepotato-and-printspoofer.md)
{% endcontent-ref %}

{% content-ref url="../juicypotato.md" %}
[juicypotato.md](../juicypotato.md)
{% endcontent-ref %}

### SeAssignPrimaryPrivilege (3.1.2)

यह **SeImpersonatePrivilege** के बहुत ही समान है, यह एक **विशेषाधिकार प्राप्त करने के लिए समान विधि** का उपयोग करेगा।\
फिर, यह विशेषाधिकार एक नई/सस्पेंड की गई प्रक्रिया के लिए **प्राथमिक टोकन का सौंपाण** करने की अनुमति देता है। विशेषाधिकृत अनुकरण टोकन के साथ आप एक प्राथमिक टोकन (DuplicateTokenEx) निकाल सकते हैं।\
टोकन के साथ, आप 'CreateProcessAsUser' के साथ एक **नई प्रक्रिया** बना सकते हैं या एक प्रक्रिया सस्पेंड कर सकते हैं और **टोकन सेट** कर सकते हैं (सामान्यत: चल रही प्रक्रिया का प्राथमिक टोकन संशोधित नहीं कर सकते हैं)।

### SeTcbPrivilege (3.1.3)

यदि आपने इस टोकन को सक्षम किया है तो आप **KERB\_S4U\_LOGON** का उपयोग करके किसी भी अन्य उपयोगकर्ता के लिए एक **अनुकरण टोकन** प्राप्त कर सकते हैं बिना पासवर्ड जाने, **किसी भी विचारात्मक समूह** (व्यवस्थापक) को जोड़ सकते हैं, टोकन के **अखंडता स्तर** को "**मध्यम**" में सेट कर सकते हैं, और इस टोकन को **वर्तमान धागे** (SetThreadToken) को सौंप सकते हैं।

### SeBackupPrivilege (3.1.4)

यह विशेषाधिकार प्रणाली को सभी पढ़ने की अनुमति देता है किसी भी फ़ाइल को (केवल पढ़ने के लिए)।\
इसका उपयोग करें **स्थानीय प्रशासक** खातों के पासवर्ड हैश पढ़ने के लिए रजिस्ट्री से और फिर हैश के साथ "**psexec**" या "**wmicexec**" का उपयोग करें।\
यह हमला काम नहीं करेगा अगर स्थानीय प्रशासक अक्षम है, या अगर यह विन्यासित है कि एक स्थानीय व्यवस्थापक यदि वह दूरस्थ रूप से कनेक्ट है तो वह व्यवस्थापक नहीं है।\
आप इस विशेषाधिकार का दुरुपयोग कर सकते हैं:

* [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)
* [https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)
* **IppSec** का पालन करें [https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec](https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec)
* या जैसा कि **बैकअप ऑपरेटर्स के साथ विशेषाधिकारों को बढ़ाने** के खंड में स्पष्ट किया गया है:

{% content-ref url="../../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### SeRestorePrivilege (3.1.5)

सिस्टम पर किसी भी फ़ाइल को **लिखने की अनुमति** देता है, फ़ाइलों के ACL के बावजूद।\
आप **सेवाओं को संशोधित** कर सकते हैं, DLL Hijacking, **डीबगर सेट** कर सकते हैं (इमेज फ़ाइल एक्जीक्यूशन विकल्प)… उच्चारण करने के लिए कई विकल्प हैं।

### SeCreateTokenPrivilege (3.1.6)

यह टोकन **केवल** उस समय **उपयोग किया जा सकता है** जब उपयोगकर्ता **टोकन का अनुकरण कर सकता है** (यहाँ तक कि SeImpersonatePrivilege के बिना भी)।\
संभावित स्थिति में, एक उपयोगकर्ता टोकन का अनुकरण कर सकता है अगर यह उसी उपयोगकर्ता के लिए है और अखंडता स्तर वर्तमान प्रक्रिया के अखंडता स्तर से कम या उसके बराबर है।\
इस मामले में, उपयोगकर्ता एक अनुकरण टोकन बना सकता है और इसमें एक विशेषाधिकृत समूह SID जोड़ सकता है।

### SeLoadDriverPrivilege (3.1.7)

**डिवाइस ड्राइवर्स लोड और अनलोड करें।**\
आपको ImagePath और Type के मान वाले रजिस्ट्री में एक प्रविष्टि बनानी होगी।\
आपको HKLM में लिखने का अधिकार नहीं है, इसलिए आपको **HKCU का उपयोग** करना होगा। लेकिन HKCU कर्नल के लिए कुछ नहीं है, कर्नल को यहाँ मार्गदर्शन देने और ड्राइवर कॉन्फ़िग के लिए अपेक्षित पथ का उपयोग करने का तरीका है: "\Registry\User\S-1-5-21-582075628-3447520101
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
### SeDebugPrivilege (3.1.9)

यह धारक को **एक और प्रक्रिया को डीबग** करने की अनुमति देता है, इसमें उस **प्रक्रिया की मेमोरी** को पढ़ना और **लिखना** शामिल है।\
इस अधिकार के साथ उपयोग किए जा सकने वाले कई विभिन्न **मेमोरी इंजेक्शन** रणनीतियाँ हैं जो अधिकांश AV/HIPS समाधानों से बचने में सक्षम हैं।

#### मेमोरी डंप

इस अधिकार के **दुरुपयोग** का एक उदाहरण है [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) को [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) से चलाना **प्रक्रिया मेमोरी को डंप** करने के लिए। उदाहरण के लिए, **स्थानीय सुरक्षा प्राधिकरण उपसिद्ध सेवा (**[**LSASS**](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service)**)** प्रक्रिया, जो एक उपयोगकर्ता सिस्टम पर लॉग ऑन करने के बाद उपयोगकर्ता क्रेडेंशियल संग्रहित करता है।

फिर आप इस डंप को मिमीकेट्ज में लोड करके पासवर्ड प्राप्त कर सकते हैं:
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
## विशेषाधिकारों की जांच
```
whoami /priv
```
**टोकन जो अक्षम दिखाई देते हैं** को सक्रिय किया जा सकता है, आप _सक्रिय_ और _अक्षम_ टोकन का दुरुपयोग कर सकते हैं।

### सभी टोकन को सक्रिय करें

आप [**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) स्क्रिप्ट का उपयोग कर सकते हैं:
```powershell
.\EnableAllTokenPrivs.ps1
whoami /priv
```
या **स्क्रिप्ट** इस [**पोस्ट**](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/) में एम्बेड करें।

## तालिका

पूर्ण टोकन विशेषाधिकार चीटशीट पर [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin), सारांश नीचे केवल विशेषाधिकार का शोषण करने के सीधे तरीके सूचीत करेगा जिससे एडमिन सत्र प्राप्त किया जा सकता है या संवेदनशील फ़ाइलें पढ़ी जा सकती हैं।\\

| विशेषाधिकार              | प्रभाव      | उपकरण                    | निष्पादन पथ                                                                                                                                                                                                                                                                                                                                     | टिप्पणियाँ                                                                                                                                                                                                                                                                                                                        |
| -------------------------- | ----------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`SeAssignPrimaryToken`** | _**एडमिन**_ | तृतीय पक्ष उपकरण          | _"यह एक उपयोगकर्ता को टोकन का अनुकरण करने और उपकरण जैसे कि potato.exe, rottenpotato.exe और juicypotato.exe का उपयोग करके nt सिस्टम में प्राइवेस्क करने की अनुमति देगा"_                                                                                                                                                                                                      | अपडेट के लिए धन्यवाद [Aurélien Chalot](https://twitter.com/Defte\_)। मैं जल्द ही इसे कुछ और रेसिपी जैसा बनाने की कोशिश करूंगा।                                                                                                                                                                                        |
| **`SeBackup`**             | **खतरा**   | _**बिल्ट-इन कमांड्स**_ | `robocopy /b` के साथ संवेदनशील फ़ाइलें पढ़ें।                                                                                                                                                                                                                                                                                                             | <p>- यदि आप %WINDIR%\MEMORY.DMP पढ़ सकते हैं तो यह और भी दिलचस्प हो सकता है<br><br>- <code>SeBackupPrivilege</code> (और robocopy) खुली फ़ाइलों के मामले में सहायक नहीं है।<br><br>- Robocopy को /b पैरामीटर के साथ काम करने के लिए दोनों SeBackup और SeRestore की आवश्यकता है।</p>                                                                      |
| **`SeCreateToken`**        | _**एडमिन**_ | तृतीय पक्ष उपकरण          | `NtCreateToken` के साथ स्थानिक एडमिन अधिकारों सहित विविध टोकन बनाएं।                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                                                |
| **`SeDebug`**              | _**एडमिन**_ | **PowerShell**          | `lsass.exe` टोकन की डुप्लिकेट बनाएं।                                                                                                                                                                                                                                                                                                                   | [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1) पर स्क्रिप्ट देखें                                                                                                                                                                                                         |
| **`SeLoadDriver`**         | _**एडमिन**_ | तृतीय पक्ष उपकरण          | <p>1. <code>szkg64.sys</code> जैसे बगी कर्नेल ड्राइवर लोड करें<br>2. ड्राइवर की कमजोरी का शोषण करें<br><br>वैकल्पिक रूप से, यह विशेषाधिकार <code>ftlMC</code> इनबिल्ट कमांड का उपयोग करके सुरक्षा संबंधित ड्राइवरों को अनलोड करने के लिए उपयोग किया जा सकता है। जैसे: <code>fltMC sysmondrv</code></p>                                                                           | <p>1. <code>szkg64</code> कमजोरी को <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732">CVE-2018-15732</a> के रूप में सूचीबद्ध किया गया है<br>2. <code>szkg64</code> <a href="https://www.greyhathacker.net/?p=1025">एक्सप्लॉइट कोड</a> को <a href="https://twitter.com/parvezghh">Parvez Anwar</a> ने बनाया है</p> |
| **`SeRestore`**            | _**एडमिन**_ | **PowerShell**          | <p>1. SeRestore विशेषाधिकार के साथ PowerShell/ISE लॉन्च करें।<br>2. <a href="https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1">Enable-SeRestorePrivilege</a> के साथ विशेषाधिकार सक्रिय करें।<br>3. utilman.exe को utilman.old में नाम बदलें<br>4. cmd.exe को utilman.exe में नाम बदलें<br>5. कंसोल लॉक करें और Win+U दबाएं</p> | <p>कुछ AV सॉफ़्टवेयर द्वारा हमला पहचाना जा सकता है।</p><p>वैकल्पिक विधि समान विशेषाधिकार का उपयोग करके "प्रोग्राम फ़ाइल्स" में स्टोर की गई सेवा बाइनरी को बदलने पर निर्भर करती है</p>                                                                                                                                                            |
| **`SeTakeOwnership`**      | _**एडमिन**_ | _**बिल्ट-इन कमांड्स**_ | <p>1. <code>takeown.exe /f "%windir%\system32"</code><br>2. <code>icalcs.exe "%windir%\system32" /grant "%username%":F</code><br>3. cmd.exe को utilman.exe में नाम बदलें<br>4. कंसोल लॉक करें और Win+U दबाएं</p>                                                                                                                                       | <p>कुछ AV सॉफ़्टवेयर द्वारा हमला पहचाना जा सकता है।</p><p>वैकल्पिक विधि समान विशेषाधिकार का उपयोग करके "प्रोग्राम फ़ाइल्स" में स्टोर की गई सेवा बाइनरी को बदलने पर निर्भर करती है।</p>                                                                                                                                                           |
| **`SeTcb`**                | _**एडमिन**_ | तृतीय पक्ष उपकरण          | <p>स्थानिक एडमिन अधिकारों को शामिल करने के लिए टोकन को मानिपुलेट करें। SeImpersonate की आवश्यकता हो सकती है।</p><p>सत्यापित करने के लिए।</p>                                                                                                                                                                                                                                     |                                                                                                                                                                                                                                                                                                                                |

## संदर्भ

* विंडोज टोकन को परिभाषित करने वाली इस तालिका को देखें: [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)
* टोकन के साथ privesc के बारे में [**इस पेपर**](https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt) को देखें।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण को डाउनलोड करना चाहते हैं या HackTricks को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके**।

</details>
