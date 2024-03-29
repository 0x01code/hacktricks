# Записний шлях Sys + Підвищення привілеїв за допомогою Dll Hijacking

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

## Вступ

Якщо ви виявили, що ви можете **записувати в папку Шляху системи** (зверніть увагу, що це не працюватиме, якщо ви можете записувати в папку Шляху користувача), це може означати, що ви можете **підвищити привілеї** в системі.

Для цього ви можете використати **Dll Hijacking**, де ви будете **захоплювати бібліотеку, яку завантажує** служба або процес з **більшими привілеями**, ніж у вас, і через те, що ця служба завантажує Dll, який, ймовірно, навіть не існує в усій системі, вона спробує завантажити його з Шляху системи, де ви можете записувати.

Для отримання додаткової інформації про **що таке Dll Hijacking** перегляньте:

{% content-ref url="../dll-hijacking.md" %}
[dll-hijacking.md](../dll-hijacking.md)
{% endcontent-ref %}

## Підвищення привілеїв за допомогою Dll Hijacking

### Пошук відсутньої Dll

Перше, що вам потрібно зробити, це **ідентифікувати процес**, який працює з **більшими привілеями**, ніж у вас, і намагається **завантажити Dll з Шляху системи**, в який ви можете записувати.

Проблема в тому, що, ймовірно, ці процеси вже працюють. Щоб знайти, які Dll відсутні у службах, вам потрібно запустити procmon якомога швидше (до завантаження процесів). Отже, щоб знайти відсутні .dll, виконайте:

* **Створіть** папку `C:\privesc_hijacking` та додайте шлях `C:\privesc_hijacking` до **змінної середовища Шляху системи**. Це можна зробити **вручну** або за допомогою **PS**:
```powershell
# Set the folder path to create and check events for
$folderPath = "C:\privesc_hijacking"

# Create the folder if it does not exist
if (!(Test-Path $folderPath -PathType Container)) {
New-Item -ItemType Directory -Path $folderPath | Out-Null
}

# Set the folder path in the System environment variable PATH
$envPath = [Environment]::GetEnvironmentVariable("PATH", "Machine")
if ($envPath -notlike "*$folderPath*") {
$newPath = "$envPath;$folderPath"
[Environment]::SetEnvironmentVariable("PATH", $newPath, "Machine")
}
```
* Запустіть **`procmon`** та перейдіть до **`Options`** --> **`Enable boot logging`** та натисніть **`OK`** у вікні підтвердження.
* Після цього **перезавантажте** систему. Після перезавантаження **`procmon`** почне **записувати** події негайно.
* Після того, як **Windows** буде **запущено, виконайте `procmon`** знову, він повідомить вас, що він працював, і запитає, чи хочете ви **зберегти** події у файл. Скажіть **так** та **збережіть події у файл**.
* **Після** того, як **файл** буде **створено**, **закрийте** відкрите вікно **`procmon`** та **відкрийте файл подій**.
* Додайте ці **фільтри**, і ви знайдете всі Dll, які **спробував завантажити** якийсь **процес** з записуваної папки **System Path**:

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### Пропущені Dlls

Виконуючи це на безкоштовній **віртуальній (vmware) машині з Windows 11**, я отримав такі результати:

<figure><img src="../../../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>

У цьому випадку .exe є непотрібними, тому ігноруйте їх, пропущені DLL були з:

| Служба                         | Dll                | CMD line                                                             |
| ------------------------------- | ------------------ | -------------------------------------------------------------------- |
| Планувальник завдань (Schedule)       | WptsExtensions.dll | `C:\Windows\system32\svchost.exe -k netsvcs -p -s Schedule`          |
| Служба діагностики політики (DPS) | Unknown.DLL        | `C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork -p -s DPS` |
| ???                             | SharedRes.dll      | `C:\Windows\system32\svchost.exe -k UnistackSvcGroup`                |

Після знаходження цього, я знайшов цей цікавий блог-пост, який також пояснює, як [**зловживати WptsExtensions.dll для підвищення привілеїв**](https://juggernaut-sec.com/dll-hijacking/#Windows\_10\_Phantom\_DLL\_Hijacking\_-\_WptsExtensionsdll). Це те, що **ми збираємося зробити зараз**.

### Експлуатація

Таким чином, для **підвищення привілеїв** ми збираємося перехопити бібліотеку **WptsExtensions.dll**. Маючи **шлях** та **ім'я**, нам просто потрібно **створити зловмисний dll**.

Ви можете [**спробувати використати будь-який з цих прикладів**](../dll-hijacking.md#creating-and-compiling-dlls). Ви можете запускати вразливості, такі як: отримати обернену оболонку, додати користувача, виконати маяк...

{% hint style="warning" %}
Зверніть увагу, що **не всі служби працюють** з **`NT AUTHORITY\SYSTEM`**, деякі також працюють з **`NT AUTHORITY\LOCAL SERVICE`**, який має **менше привілеїв**, і ви **не зможете створити нового користувача**, використовуючи його дозволи.\
Однак у цього користувача є привілеї **`seImpersonate`**, тому ви можете використовувати [**набір інструментів potato для підвищення привілеїв**](../roguepotato-and-printspoofer.md). Таким чином, у цьому випадку обернена оболонка є кращим варіантом, ніж спроба створення користувача.
{% endhint %}

На момент написання служба **Планувальник завдань** працює з **Nt AUTHORITY\SYSTEM**.

Після **створення зловмисного Dll** (_у моєму випадку я використовував x64 обернену оболонку і отримав оболонку, але захисник вбив її, оскільки вона була від msfvenom_), збережіть її в записувану папку **System Path** з ім'ям **WptsExtensions.dll** та **перезапустіть** комп'ютер (або перезапустіть службу або виконайте будь-що, щоб перезапустити порушену службу/програму).

Коли служба буде перезапущена, **dll повинен бути завантажений та виконаний** (ви можете **повторно використовувати** трюк з **procmon**, щоб перевірити, чи **була бібліотека завантажена, як очікувалося**).
