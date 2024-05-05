# Ucieczka z KIOSKów

<details>

<summary><strong>Nauka hakowania AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakowania, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to silnik wyszukiwania zasilany **dark webem**, który oferuje **darmowe** funkcje sprawdzania, czy firma lub jej klienci zostali **skompromitowani** przez **złośliwe oprogramowanie kradnące informacje**.

Ich głównym celem WhiteIntel jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz sprawdzić ich stronę internetową i wypróbować ich silnik za **darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

***

## Sprawdź możliwe działania wewnątrz aplikacji GUI

**Dialogi ogólne** to te opcje związane z **zapisywaniem pliku**, **otwieraniem pliku**, wybieraniem czcionki, koloru... Większość z nich będzie **oferować pełną funkcjonalność Eksploratora**. Oznacza to, że będziesz mógł uzyskać dostęp do funkcji Eksploratora, jeśli będziesz mógł uzyskać dostęp do tych opcji:

* Zamknij/Zamknij jako
* Otwórz/Otwórz za pomocą
* Drukuj
* Eksportuj/Importuj
* Szukaj
* Skanuj

Powinieneś sprawdzić, czy możesz:

* Modyfikować lub tworzyć nowe pliki
* Tworzyć dowiązania symboliczne
* Uzyskać dostęp do obszarów ograniczonych
* Wykonywać inne aplikacje

### Wykonanie polecenia

Być może **korzystając z opcji `Otwórz za pomocą`** możesz otworzyć/wywołać pewnego rodzaju powłokę.

#### Windows

Na przykład _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ znajdź więcej binarnych plików, które można użyć do wykonywania poleceń (i wykonywania nieoczekiwanych działań) tutaj: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

#### \*NIX \_\_

_bash, sh, zsh..._ Więcej tutaj: [https://gtfobins.github.io/](https://gtfobins.github.io)

## Windows

### Omijanie ograniczeń ścieżki

* **Zmienne środowiskowe**: Istnieje wiele zmiennych środowiskowych wskazujących na pewną ścieżkę
* **Inne protokoły**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Dowiązania symboliczne**
* **Skróty klawiszowe**: CTRL+N (otwórz nową sesję), CTRL+R (Wykonaj polecenia), CTRL+SHIFT+ESC (Menedżer zadań), Windows+E (otwórz Eksplorator), CTRL-B, CTRL-I (Ulubione), CTRL-H (Historia), CTRL-L, CTRL-O (Plik/Otwórz dialogowy), CTRL-P (Dialog drukowania), CTRL-S (Zapisz jako)
* Ukryte menu administracyjne: CTRL-ALT-F8, CTRL-ESC-F9
* **URI powłoki**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **Ścieżki UNC**: Ścieżki do łączenia się z udostępnionymi folderami. Powinieneś spróbować połączyć się z C$ lokalnej maszyny ("\\\127.0.0.1\c$\Windows\System32")
* **Więcej ścieżek UNC:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      |

### Pobierz swoje binaria

Konsola: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Eksplorator: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Edytor rejestru: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

### Dostęp do systemu plików z przeglądarki

| ŚCIEŻKA                | ŚCIEŻKA              | ŚCIEŻKA               | ŚCIEŻKA                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |

### Skróty

* Sticky Keys – Naciśnij SHIFT 5 razy
* Mouse Keys – SHIFT+ALT+NUMLOCK
* High Contrast – SHIFT+ALT+PRINTSCN
* Toggle Keys – Przytrzymaj NUMLOCK przez 5 sekund
* Filter Keys – Przytrzymaj prawy SHIFT przez 12 sekund
* WINDOWS+F1 – Wyszukiwanie w systemie Windows
* WINDOWS+D – Pokaż pulpit
* WINDOWS+E – Uruchom Eksploratora Windows
* WINDOWS+R – Uruchom
* WINDOWS+U – Centrum ułatwień dostępu
* WINDOWS+F – Wyszukiwanie
* SHIFT+F10 – Menu kontekstowe
* CTRL+SHIFT+ESC – Menedżer zadań
* CTRL+ALT+DEL – Ekran powitalny w nowszych wersjach systemu Windows
* F1 – Pomoc F3 – Szukaj
* F6 – Pasek adresu
* F11 – Przełącz pełny ekran w przeglądarce Internet Explorer
* CTRL+H – Historia przeglądarki Internet Explorer
* CTRL+T – Internet Explorer – Nowa karta
* CTRL+N – Internet Explorer – Nowa strona
* CTRL+O – Otwórz plik
* CTRL+S – Zapisz CTRL+N – Nowe RDP / Citrix
### Swipy

* Przesuń palcem z lewej strony na prawą, aby zobaczyć wszystkie otwarte okna, minimalizując aplikację KIOSK i uzyskując bezpośredni dostęp do całego systemu operacyjnego;
* Przesuń palcem z prawej strony na lewą, aby otworzyć Centrum akcji, minimalizując aplikację KIOSK i uzyskując bezpośredni dostęp do całego systemu operacyjnego;
* Przesuń palcem z górnej krawędzi, aby sprawić, że pasek tytułu będzie widoczny dla aplikacji otwartej w trybie pełnoekranowym;
* Przesuń palcem w górę z dołu, aby pokazać pasek zadań w aplikacji pełnoekranowej.

### Triki z Internet Explorera

#### 'Pasek narzędzi obrazu'

To pasek narzędzi, który pojawia się w lewym górnym rogu obrazu po jego kliknięciu. Będziesz mógł zapisać, wydrukować, wysłać e-mail, otworzyć "Moje obrazy" w Eksploratorze. Kiosk musi korzystać z przeglądarki Internet Explorer.

#### Protokół powłoki

Wpisz te adresy URL, aby uzyskać widok Eksploratora:

* `shell:Narzędzia administracyjne`
* `shell:Biblioteka dokumentów`
* `shell:Biblioteki`
* `shell:ProfileUżytkownika`
* `shell:Osobiste`
* `shell:FolderDomowyWyszukiwania`
* `shell:FolderMiejscSieciowych`
* `shell:WyślijDo`
* `shell:ProfileUżytkownika`
* `shell:Wspólne narzędzia administracyjne`
* `shell:MójKomputer`
* `shell:Internet`
* `Shell:Profil`
* `Shell:PlikiProgramów`
* `Shell:System`
* `Shell:FolderPaneluSterowania`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Panel sterowania
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> Mój Komputer
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> Moje miejsca sieciowe
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

### Pokaż rozszerzenia plików

Sprawdź tę stronę, aby uzyskać więcej informacji: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

## Triki przeglądarek

Kopia zapasowa wersji iKat:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\\

Utwórz wspólny dialog za pomocą JavaScript i uzyskaj dostęp do eksploratora plików: `document.write('<input/type=file>')`\
Źródło: https://medium.com/@Rend\_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

## iPad

### Gesty i przyciski

* Przesuń w górę czterema (lub pięcioma) palcami / Podwójne stuknięcie przycisku Home: Aby zobaczyć widok wielozadaniowy i zmienić aplikację
* Przesuń palcem w jedną lub drugą stronę czterema lub pięcioma palcami: Aby przejść do następnej/poprzedniej aplikacji
* Szczypnij ekran pięcioma palcami / Dotknij przycisku Home / Przesuń w górę jednym palcem z dołu ekranu w szybkim ruchu do góry: Aby uzyskać dostęp do ekranu głównego
* Przesuń palcem z dołu ekranu zaledwie 1-2 cali (wolno): Doker się pojawi
* Przesuń w dół z górnej krawędzi ekranu jednym palcem: Aby zobaczyć powiadomienia
* Przesuń w dół jednym palcem w prawym górnym rogu ekranu: Aby zobaczyć centrum sterowania iPad Pro
* Przesuń palcem z lewej krawędzi ekranu 1-2 cali: Aby zobaczyć widok Dzisiaj
* Szybko przesuń palcem z centrum ekranu w prawo lub w lewo: Aby przejść do następnej/poprzedniej aplikacji
* Naciśnij i przytrzymaj przycisk Wł./Wył./Uśpij w prawym górnym rogu iPada + Przesuń suwak Wyłącz do końca w prawo: Aby wyłączyć zasilanie
* Naciśnij przycisk Wł./Wył./Uśpij w prawym górnym rogu iPada i przycisk Home przez kilka sekund: Aby wymusić twardy reset
* Naciśnij przycisk Wł./Wył./Uśpij w prawym górnym rogu iPada i przycisk Home szybko: Aby zrobić zrzut ekranu, który pojawi się w lewym dolnym rogu ekranu. Naciśnij oba przyciski jednocześnie bardzo krótko, jakbyś je przytrzymywał kilka sekund, twardy reset zostanie wykonany.
#### Skróty klawiszowe w Safari

| Skrót                | Działanie                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (Command-L)          | Otwórz lokalizację                                    |
| ⌘T                      | Otwórz nową kartę                                   |
| ⌘W                      | Zamknij bieżącą kartę                            |
| ⌘R                      | Odśwież bieżącą kartę                          |
| ⌘.                      | Zatrzymaj ładowanie bieżącej karty                     |
| ^⇥                      | Przełącz się na następną kartę                           |
| ^⇧⇥ (Control-Shift-Tab) | Przejdź do poprzedniej karty                         |
| ⌘L                      | Wybierz pole tekstowe/URL, aby je zmodyfikować     |
| ⌘⇧T (Command-Shift-T)   | Otwórz ostatnio zamkniętą kartę (można użyć kilka razy) |
| ⌘\[                     | Wróć do poprzedniej strony w historii przeglądania      |
| ⌘]                      | Przejdź do przodu o jedną stronę w historii przeglądania   |
| ⌘⇧R                     | Aktywuj tryb czytnika                             |

#### Skróty klawiszowe w Mail

| Skrót                   | Działanie                       |
| -------------------------- | ---------------------------- |
| ⌘L                         | Otwórz lokalizację                |
| ⌘T                         | Otwórz nową kartę               |
| ⌘W                         | Zamknij bieżącą kartę        |
| ⌘R                         | Odśwież bieżącą kartę      |
| ⌘.                         | Zatrzymaj ładowanie bieżącej karty |
| ⌘⌥F (Command-Option/Alt-F) | Szukaj w swojej skrzynce odbiorczej       |

## Odnośniki

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to wyszukiwarka zasilana **dark-web**, która oferuje **darmowe** funkcje sprawdzania, czy firma lub jej klienci zostali **skompromitowani** przez **złośliwe oprogramowanie kradnące informacje**.

Ich głównym celem WhiteIntel jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz odwiedzić ich stronę internetową i wypróbować ich silnik za **darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLAN SUBSKRYPCYJNY**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**Grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
