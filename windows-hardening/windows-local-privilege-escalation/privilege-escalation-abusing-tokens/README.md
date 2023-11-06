# टोकन का दुरुपयोग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में।**

</details>

## टोकन

यदि आपको **पता नहीं है कि Windows एक्सेस टोकन्स क्या होते हैं**, तो आगे बढ़ने से पहले इस पेज को पढ़ें:

{% content-ref url="../access-tokens.md" %}
[access-tokens.md](../access-tokens.md)
{% endcontent-ref %}

**शायद आप उन टोकन्स का दुरुपयोग करके विशेषाधिकारों को बढ़ा सकते हो**

### SeImpersonatePrivilege (3.1.1)

इस विशेषाधिकार को रखने वाली कोई प्रक्रिया किसी भी **टोकन** को **अनुकरण** कर सकती है (लेकिन नया टोकन नहीं बना सकती है) जिसके लिए वह संबंधित हैंडल प्राप्त कर सकती है। आप एक **विशेषाधिकारी टोकन** प्राप्त कर सकते हैं एक **Windows सेवा** (DCOM) से (एनटीएलएम प्रमाणीकरण के माध्यम से) जब वह एक्सप्लॉइट के खिलाफ एक **NTLM प्रमाणीकरण** करती है, फिर **SYSTEM** के रूप में एक प्रक्रिया को निष्पादित करती है। [juicy-potato](https://github.com/ohpe/juicy-potato), [RogueWinRM ](https://github.com/antonioCoco/RogueWinRM)(winrm अक्षम होना चाहिए), [SweetPotato](https://github.com/CCob/SweetPotato), [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) के साथ इसे एक्सप्लॉइट करें:

{% content-ref url="../roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](../roguepotato-and-printspoofer.md)
{% endcontent-ref %}

{% content-ref url="../juicypotato.md" %}
[juicypotato.md](../juicypotato.md)
{% endcontent-ref %}

### SeAssignPrimaryPrivilege (3.1.2)

यह **SeImpersonatePrivilege** के बहुत ही समान है, यह एकीकृत टोकन प्राप्त करने के लिए **एकीकृत विधि** का उपयोग करेगा।
फिर, यह विशेषाधिकार एक नए/सस्पेंड किए गए प्रक्रिया को **प्राथमिक टोकन सौंपने** की अनुमति देता है। विशेषाधिकारी अनुकरण टोकन के साथ आप एक प्राथमिक टोकन (DuplicateTokenEx) उत्पन्न कर सकते हैं।
टोकन के साथ, आप 'CreateProcessAsUser' के साथ एक **नई प्रक्रिया** बना सकते हैं या एक प्रक्रिया सस्पेंड करके टोकन सेट कर सकते हैं (सामान्यतः, चल रही प्रक्रिया के प्राथमिक टोकन को आप संशोधित नहीं कर सकते हैं)।

### SeTcbPrivilege (3.1.3)

यदि आपने इस टोकन को सक्षम किया है तो आप **KERB\_S4U\_LOGON** का उपयोग करके किसी भी अन्य उपयोगकर्ता के लिए **अनुकरण टोकन** प्राप्त कर सकते हैं जबकि प्रमाण-पत्र को नहीं जानते हैं, टोकन में एक **विचित्र समूह** (व्यवस्थापक) जोड़ सकते हैं, टोकन के **अखंडता स्तर** को "**मध्यम**" में सेट कर सकते हैं, और इस टोकन को **वर्तमान धागे** (SetThreadToken) को सौंप सकते हैं।

### SeBackupPrivilege (3.1.4)

यह विशेषाधिकार सिस्टम को किसी भी फ़ाइल के लिए **सभी पढ़ने की** अनुमति निर्धारित करता है (केवल पढ़ने के ल
### SeRestorePrivilege (3.1.5)

**लिखने की पहुंच** सिस्टम पर किसी भी फ़ाइल को, फ़ाइल की ACL के बावजूद, होती है।\
आप **सेवाओं को संशोधित कर सकते हैं**, DLL Hijacking कर सकते हैं, **डीबगर** सेट कर सकते हैं (Image File Execution Options)... इसमें बढ़ने के लिए कई विकल्प हैं।

### SeCreateTokenPrivilege (3.1.6)

यह टोकन **उपयोग किया जा सकता है** EoP विधि के रूप में **केवल** तभी जब उपयोगकर्ता **टोकन का अनुकरण कर सकता है** (यहां तक कि SeImpersonatePrivilege के बिना भी)।\
एक संभावित परिदृश्य में, यदि उपयोगकर्ता टोकन का अनुकरण कर सकता है और यह वही उपयोगकर्ता के लिए है और अवश्यकता स्तर वर्तमान प्रक्रिया के स्तर से कम या बराबर है।\
इस मामले में, उपयोगकर्ता **एक अनुकरण टोकन बना सकता है** और इसमें एक विशेषाधिकारी समूह SID जोड़ सकता है।

### SeLoadDriverPrivilege (3.1.7)

**डिवाइस ड्राइवर लोड और अनलोड करें।**\
आपको ImagePath और Type के लिए रजिस्ट्री में एक प्रविष्टि बनानी होगी।\
आपको HKLM में लिखने की पहुंच नहीं है, इसलिए आपको **HKCU का उपयोग करना होगा**। लेकिन HKCU कर्नल के लिए कुछ नहीं होता है, यहां कर्नल को निर्देशित करने और ड्राइवर कॉन्फ़िगरेशन के लिए अपेक्षित पथ का उपयोग करने का तरीका है: "\Registry\User\S-1-5-21-582075628-3447520101-2530640108-1003\System\CurrentControlSet\Services\DriverName" (वर्तमान उपयोगकर्ता का **RID** है)।\
इसलिए, आपको **HKCU में उस पूरे पथ को बनाना होगा और ImagePath** (बाइनरी का पथ जो निष्पादित होने जा रहा है) **और Type** (SERVICE\_KERNEL\_DRIVER 0x00000001) सेट करना होगा।

{% content-ref url="abuse-seloaddriverprivilege.md" %}
[abuse-seloaddriverprivilege.md](abuse-seloaddriverprivilege.md)
{% endcontent-ref %}

### SeTakeOwnershipPrivilege (3.1.8)

यह विशेषाधिकार बहुत समान **SeRestorePrivilege** के साथ है।\
यह एक प्रक्रिया को "विचारधीन पहुंच के बिना एक ऑब्जेक्ट के स्वामित्व को ले सकने" की अनुमति देकर WRITE\_OWNER पहुंच अधिकार को प्रदान करके होता है।\
सबसे पहले, आपको रजिस्ट्री कुंजी का स्वामित्व **ले लेना होगा** जिस पर आप लिखने जा रहे हैं और इसे **DACL को संशोधित** करना होगा ताकि आप इस पर लिख सकें।
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

इसे धारक को दूसरे प्रक्रिया को **डीबग करने** की अनुमति देता है, जिसमें इस प्रक्रिया की मेमोरी में **पढ़ने और लिखने** की शामिल होती है।\
इस अनुमति के साथ इस्तेमाल किए जाने वाले विभिन्न **मेमोरी इंजेक्शन** रणनीतियाँ हैं जो अधिकांश AV/HIPS समाधानों को टाल सकती हैं।

#### मेमोरी डंप

इस अनुमति के **दुरुपयोग** का एक उदाहरण है [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) को [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) से चलाना और एक प्रक्रिया की मेमोरी को **डंप करना**। उदाहरण के लिए, **स्थानीय सुरक्षा प्राधिकरण उपस्थिति उपस्थापन सेवा (**[**LSASS**](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service)**)** प्रक्रिया, जो उपयोगकर्ता श्रेणीपत्रों को एक प्रणाली में लॉग ऑन करने के बाद संग्रहीत करती है।

फिर आप मिमीकेट्ज़ में इस डंप को लोड करके पासवर्ड प्राप्त कर सकते हैं:
```
mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```
#### RCE

यदि आप `NT SYSTEM` शेल प्राप्त करना चाहते हैं, तो आप निम्नलिखित का उपयोग कर सकते हैं:

* ****[**SeDebugPrivilegePoC**](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC)****
* ****[**psgetsys.ps1**](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1)****
```powershell
# Get the PID of a process running as NT SYSTEM
import-module psgetsys.ps1; [MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>)
```
## विशेषाधिकारों की जांच करें

To check the privileges of a user, you can use the following methods:

### 1. Using the `whoami` command

The `whoami` command displays the username of the current user. By default, it also shows the group memberships and privileges associated with the user.

```plaintext
whoami /priv
```

### 2. Using the `net user` command

The `net user` command provides detailed information about a user account, including the group memberships and privileges.

```plaintext
net user <username>
```

### 3. Using the `whoami /all` command

The `whoami /all` command displays detailed information about the current user, including the security privileges and group memberships.

```plaintext
whoami /all
```

### 4. Using the `secpol.msc` GUI

You can also use the `secpol.msc` GUI to check the privileges of a user. Follow these steps:

1. Open the "Local Security Policy" by searching for `secpol.msc` in the Start menu.
2. In the left pane, navigate to "Security Settings" > "Local Policies" > "User Rights Assignment".
3. In the right pane, you will find a list of user rights and the users or groups assigned to them.

By checking the privileges of a user, you can identify any potential vulnerabilities or opportunities for privilege escalation.
```
whoami /priv
```
वे **टोकन जो अक्षम दिखाई देते हैं** को सक्षम किया जा सकता है, आप वास्तव में _सक्षम_ और _अक्षम_ टोकन का दुरुपयोग कर सकते हैं।

### सभी टोकनों को सक्षम करें

आप [**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) स्क्रिप्ट का उपयोग करके सभी टोकनों को सक्षम कर सकते हैं:
```powershell
.\EnableAllTokenPrivs.ps1
whoami /priv
```
या इस [पोस्ट](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/) में एम्बेड किया गया **स्क्रिप्ट**।

## तालिका

पूर्ण टोकन विशेषाधिकार चीटशीट [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin) पर उपलब्ध है, नीचे सारांश में केवल प्रशासक सत्र या संवेदनशील फ़ाइलों को पढ़ने के लिए विशेषाधिकार का उपयोग करने के सीधे तरीकों की सूची दी गई है।

| विशेषाधिकार                | प्रभाव       | उपकरण                   | निष्पादन पथ                                                                                                                                                                                                                                                                                                                                     | टिप्पणियाँ                                                                                                                                                                                                                                                                                                                    |
| -------------------------- | ----------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`SeAssignPrimaryToken`** | _**व्यवस्थापक**_ | तृतीय पक्ष उपकरण          | _"इससे उपयोगकर्ता टोकन की अनुकरण करने और उपकरणों जैसे potato.exe, rottenpotato.exe और juicypotato.exe का उपयोग करके nt सिस्टम में प्रवेश करने की अनुमति होगी"_                                                                                                                                                                                                      | अद्यतन के लिए धन्यवाद [Aurélien Chalot](https://twitter.com/Defte_)। मैं जल्द ही इसे कुछ और रेसिपी जैसा बनाने की कोशिश करूंगा।                                                                                                                                                                                        |
| **`SeBackup`**             | **खतरा**    | _**अंतर्निहित आदेश**_ | `robocopy /b` के साथ संवेदनशील फ़ाइलें पढ़ें                                                                                                                                                                                                                                                                                                             | <p>- यदि आप %WINDIR%\MEMORY.DMP पढ़ सकते हैं तो यह और भी रोचक हो सकता है।<br><br>- <code>SeBackupPrivilege</code> (और robocopy) फ़ाइलें खोलने के लिए सहायक नहीं हैं।<br><br>- robocopy को /b पैरामीटर के साथ काम करने के लिए दोनों SeBackup और SeRestore की आवश्यकता होती है।</p>                                                                      |
| **`SeCreateToken`**        | _**व्यवस्थापक**_ | तृतीय पक्ष उपकरण          | `NtCreateToken` के साथ स्थानीय व्यवस्थापक अधिकारों के साथ विभिन्न टोकन बनाएं।                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                                                |
| **`SeDebug`**              | _**व्यवस्थापक**_ | **PowerShell**          | `lsass.exe` टोकन की नकल करें।                                                                                                                                                                                                                                                                                                                   | [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1) पर स्क्रिप्ट देखें                                                                                                                                                                                                         |
| **`SeLoadDriver`**         | _**व्यवस्थापक**_ | तृतीय पक्ष उपकरण          | <p>1. ऐसा बगीला कर्नल ड्राइवर लोड करें जैसे <code>szkg64.sys</code><br>2. ड्राइवर की कमजोरी का शोषण करें<br><br>वैकल्पिक रूप से, इस विशेषाधिकार का उपयोग सुरक्षा संबंधित ड्राइवरों को <code>ftlMC</code> इनबिल्ट आदेश के साथ अनलोड करने के लिए किया जा सकता है। उदा।: <code>fltMC sysmondrv</code></p>                                                                           | <p>1. <code>szkg64</code> कमजोरी [CVE-2018-15732](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732) के रूप में सूचीबद्ध है<br>2. <code>szkg64</code> [शोषण कोड](https://www.greyhathacker.net/?p=1025) को [Parvez Anwar](https://twitter.com/parvezghh) ने बनाया है</p> |
| **`SeRestore`**            | _**व्यवस्थापक**_ | **PowerShell**          | <p>1. SeRestore विशेषाधिकार के साथ PowerShell/ISE लॉन्च करें।<br>2. [Enable-SeRestorePrivilege](https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1) के साथ विशेषाधिकार सक्षम करें।<br>3. utilman.exe को utilman.old में नामांकित करें<br>4. cmd.exe को utilman.exe में नामांकित करें<br>5. कंसोल लॉक करें और Win+U दबाएं</p> | कुछ AV सॉफ़्टवेयर द्वारा हमला पहचाना जा सकता है।<p>वैकल्पिक तरीका उसी विशेषाधिकार का उपयोग करके "प्रोग्राम फ़ाइल" में संग्रहीत सेवा बाइनरी को बदलकर निर्धारित हो सकता है</p>                                                                                                                                                            |
| **`SeTakeOwnership`**      | _**व्यवस्थापक**_ | _**अंतर्निहित आदेश**_ | <p>1. <code>takeown.exe /f "%windir%\system32"</code><br>2. <code>icalcs.exe "%windir%\system32" /grant "%username%":F</code><br>3. cmd.exe को utilman.exe में नामांकित करें<br>4. कंसोल लॉक करें और Win+U दबाएं</p>                                                                                                                                       | कुछ AV सॉफ़्टवेयर द्वारा हमला पहचाना जा सकता है।<p>वैकल्पिक तरीका उसी विशेषाधिकार का उपयोग करके "प्रोग्राम फ़ाइल" म
