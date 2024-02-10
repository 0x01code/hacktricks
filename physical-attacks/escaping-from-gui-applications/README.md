<details>

<summary><strong>Lernen Sie AWS-Hacking von Grund auf mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks bewerben möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>


# Überprüfen möglicher Aktionen innerhalb der GUI-Anwendung

**Gängige Dialogfelder** sind Optionen zum **Speichern einer Datei**, **Öffnen einer Datei**, Auswahl einer Schriftart, einer Farbe... Die meisten von ihnen bieten eine vollständige Explorer-Funktionalität an. Dies bedeutet, dass Sie auf Explorer-Funktionen zugreifen können, wenn Sie auf diese Optionen zugreifen können:

* Schließen/Schließen als
* Öffnen/Öffnen mit
* Drucken
* Exportieren/Importieren
* Suchen
* Scannen

Sie sollten überprüfen, ob Sie Folgendes tun können:

* Dateien ändern oder neue Dateien erstellen
* Symbolische Links erstellen
* Zugriff auf eingeschränkte Bereiche erhalten
* Andere Apps ausführen

## Befehlsausführung

Vielleicht können Sie **mit der Option `Öffnen mit`** eine Art Shell öffnen/ausführen.

### Windows

Zum Beispiel _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ finden Sie weitere Binärdateien, die zum Ausführen von Befehlen (und zum Ausführen unerwarteter Aktionen) verwendet werden können, hier: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

### \*NIX __

_bash, sh, zsh..._ Mehr hier: [https://gtfobins.github.io/](https://gtfobins.github.io)

# Windows

## Umgehen von Pfadbeschränkungen

* **Umgebungsvariablen**: Es gibt viele Umgebungsvariablen, die auf einen bestimmten Pfad verweisen
* **Andere Protokolle**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Symbolische Links**
* **Verknüpfungen**: STRG+N (neue Sitzung öffnen), STRG+R (Befehle ausführen), STRG+UMSCHALT+ESC (Task-Manager), Windows+E (Explorer öffnen), STRG-B, STRG-I (Favoriten), STRG-H (Verlauf), STRG-L, STRG-O (Datei/Öffnen-Dialog), STRG-P (Drucken-Dialog), STRG-S (Speichern unter)
* Verstecktes Administrationsmenü: STRG-ALT-F8, STRG-ESC-F9
* **Shell-URIs**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **UNC-Pfade**: Pfade zum Verbinden mit freigegebenen Ordnern. Sie sollten versuchen, sich mit dem C$ der lokalen Maschine zu verbinden ("\\\127.0.0.1\c$\Windows\System32")
* **Weitere UNC-Pfade:**

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

## Laden Sie Ihre Binärdateien herunter

Konsole: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Explorer: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Registrierungseditor: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

## Zugriff auf das Dateisystem über den Browser

| PATH                | PATH              | PATH               | PATH                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |

## Verknüpfungen

* Sticky Keys – SHIFT 5 Mal drücken
* Mouse Keys – SHIFT+ALT+NUMLOCK
* Hoher Kontrast – SHIFT+ALT+DRUCK
* Umschalttasten – NUMLOCK 5 Sekunden lang gedrückt halten
* Filtertasten – Rechte SHIFT 12 Sekunden lang gedrückt halten
* WINDOWS+F1 – Windows-Suche
* WINDOWS+D – Desktop anzeigen
* WINDOWS+E – Windows Explorer starten
* WINDOWS+R – Ausführen
* WINDOWS+U – Center für erleichterten Zugriff
* WINDOWS+F – Suche
* SHIFT+F10 – Kontextmenü
* CTRL+SHIFT+ESC – Task-Manager
* CTRL+ALT+DEL – Begrüßungsbildschirm in neueren Windows-Versionen
* F1 – Hilfe F3 – Suche
* F6 – Adressleiste
* F11 – Vollbildmodus in Internet Explorer umschalten
* CTRL+H – Internet Explorer-Verlauf
* CTRL+T – Internet Explorer – Neuer Tab
* CTRL+N – Internet Explorer – Neue Seite
* CTRL+O – Datei öffnen
* CTRL+S – Speichern CTRL+N – Neue RDP / Citrix
## Wischgesten

* Wischen Sie von der linken Seite nach rechts, um alle geöffneten Fenster anzuzeigen, minimieren Sie die KIOSK-App und greifen Sie direkt auf das gesamte Betriebssystem zu.
* Wischen Sie von der rechten Seite nach links, um das Aktionscenter zu öffnen, minimieren Sie die KIOSK-App und greifen Sie direkt auf das gesamte Betriebssystem zu.
* Wischen Sie von oben, um die Titelleiste für eine im Vollbildmodus geöffnete App sichtbar zu machen.
* Wischen Sie von unten nach oben, um die Taskleiste in einer Vollbild-App anzuzeigen.

## Internet Explorer Tricks

### 'Bild-Toolbar'

Es handelt sich um eine Symbolleiste, die oben links auf dem Bild erscheint, wenn darauf geklickt wird. Sie können speichern, drucken, per E-Mail senden, "Eigene Bilder" im Explorer öffnen. Der Kiosk muss Internet Explorer verwenden.

### Shell-Protokoll

Geben Sie diese URLs ein, um eine Explorer-Ansicht zu erhalten:

* `shell:Administrative Tools`
* `shell:DocumentsLibrary`
* `shell:Libraries`
* `shell:UserProfiles`
* `shell:Personal`
* `shell:SearchHomeFolder`
* `shell:NetworkPlacesFolder`
* `shell:SendTo`
* `shell:UserProfiles`
* `shell:Common Administrative Tools`
* `shell:MyComputerFolder`
* `shell:InternetFolder`
* `Shell:Profile`
* `Shell:ProgramFiles`
* `Shell:System`
* `Shell:ControlPanelFolder`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Systemsteuerung
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> Mein Computer
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> Meine Netzwerkumgebung
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

## Dateierweiterungen anzeigen

Weitere Informationen finden Sie auf dieser Seite: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

# Browser-Tricks

Sichern Sie iKat-Versionen:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\

Erstellen Sie einen gemeinsamen Dialog mit JavaScript und greifen Sie auf den Datei-Explorer zu: `document.write('<input/type=file>')`
Quelle: https://medium.com/@Rend_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

# iPad

## Gesten und Tasten

* Wischen Sie mit vier (oder fünf) Fingern nach oben / Doppelklicken Sie auf die Home-Taste: Um die Multitasking-Ansicht anzuzeigen und die App zu wechseln

* Wischen Sie mit vier oder fünf Fingern in eine Richtung: Um zur nächsten/vorherigen App zu wechseln

* Zoomen Sie mit fünf Fingern auf dem Bildschirm / Berühren Sie die Home-Taste / Wischen Sie mit einem Finger von unten schnell nach oben: Um auf den Startbildschirm zuzugreifen

* Wischen Sie mit einem Finger von unten etwa 1-2 Zoll (langsam): Das Dock wird angezeigt

* Wischen Sie mit einem Finger von oben auf dem Display nach unten: Um Ihre Benachrichtigungen anzuzeigen

* Wischen Sie mit einem Finger von oben rechts auf dem Bildschirm nach unten: Um das Kontrollzentrum des iPad Pro anzuzeigen

* Wischen Sie mit einem Finger von links auf dem Bildschirm 1-2 Zoll: Um die Ansicht "Heute" anzuzeigen

* Wischen Sie schnell mit einem Finger von der Mitte des Bildschirms nach rechts oder links: Um zur nächsten/vorherigen App zu wechseln

* Drücken und halten Sie die Ein/**Aus**/Standby-Taste in der oberen rechten Ecke des **iPad +** Bewegen Sie den Schieberegler "Ausschalten" ganz nach rechts: Um das Gerät auszuschalten

* Drücken Sie die Ein/**Aus**/Standby-Taste in der oberen rechten Ecke des **iPad und die Home-Taste einige Sekunden lang**: Um das Gerät hart auszuschalten

* Drücken Sie die Ein/**Aus**/Standby-Taste in der oberen rechten Ecke des **iPad und die Home-Taste schnell**: Um einen Screenshot aufzunehmen, der unten links auf dem Display angezeigt wird. Drücken Sie beide Tasten gleichzeitig sehr kurz, da sonst ein hartes Ausschalten durchgeführt wird.

## Verknüpfungen

Sie sollten eine iPad-Tastatur oder einen USB-Tastaturadapter haben. Es werden nur Verknüpfungen angezeigt, die beim Verlassen der Anwendung helfen können.

| Taste | Name         |
| ----- | ------------ |
| ⌘     | Befehl       |
| ⌥     | Option (Alt) |
| ⇧     | Umschalt     |
| ↩     | Eingabe      |
| ⇥     | Tab          |
| ^     | Steuerung    |
| ←     | Linke Pfeiltaste   |
| →     | Rechte Pfeiltaste  |
| ↑     | Obere Pfeiltaste     |
| ↓     | Untere Pfeiltaste   |

### Systemverknüpfungen

Diese Verknüpfungen gelten für die visuellen Einstellungen und Soundeinstellungen, abhängig von der Verwendung des iPads.

| Verknüpfung | Aktion                                                                         |
| ----------- | ------------------------------------------------------------------------------ |
| F1          | Bildschirm dimmen                                                              |
| F2          | Bildschirm erhellen                                                            |
| F7          | Zurück zum vorherigen Song                                                      |
| F8          | Wiedergabe/Pause                                                               |
| F9          | Nächster Song                                                                  |
| F10         | Stummschalten                                                                   |
| F11         | Lautstärke verringern                                                          |
| F12         | Lautstärke erhöhen                                                             |
| ⌘ Leertaste | Eine Liste der verfügbaren Sprachen anzeigen; um eine auszuwählen, tippen Sie erneut auf die Leertaste. |

### iPad-Navigation

| Verknüpfung                                           | Aktion                                                  |
| ----------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                    | Zum Startbildschirm wechseln                            |
| ⌘⇧H (Befehl-Umschalt-H)                               | Zum Startbildschirm wechseln                            |
| ⌘ (Leertaste)                                         | Spotlight öffnen                                       |
| ⌘⇥ (Befehl-Tab)                                       | Liste der zuletzt verwendeten Apps anzeigen             |
| ⌘\~                                                   | Zur letzten App wechseln                               |
| ⌘⇧3 (Befehl-Umschalt-3)                               | Screenshot (erscheint unten links zum Speichern oder Bearbeiten) |
| ⌘⇧4                                                   | Screenshot aufnehmen und im Editor öffnen              |
| ⌘ gedrückt halten                                      | Liste der für die App verfügbaren Verknüpfungen anzeigen |
| ⌘⌥D (Befehl-Option/Alt-D)                             | Das Dock aufrufen                                      |
| ^⌥H (Steuerung-Option-H)                              | Home-Taste                                              |
| ^⌥H H (Steuerung-Option-H-H)                          | Multitasking-Leiste anzeigen                            |
| ^⌥I (Steuerung-Option-i)                              | Elementauswahl                                          |
| Escape                                                | Zurück-Schaltfläche                                     |
| → (Rechte Pfeiltaste)                                 | Nächstes Element                                        |
| ← (Linke Pfeiltaste)                                  | Vorheriges Element                                      |
| ↑↓ (Obere Pfeiltaste, Untere Pfeiltaste)              | Gleichzeitig auf ausgewähltes Element tippen            |
| ⌥ ↓ (Option-Untere Pfeiltaste)                        | Nach unten scrollen                                    |
| ⌥↑ (Option-Obere Pfeiltaste)                          | Nach oben scrollen                                     |
| ⌥← oder ⌥→ (Option-Linke Pfeiltaste oder Option-Rechte Pfeiltaste) | Nach links oder rechts scrollen                         |
| ^⌥S (Steuerung-Option-S)                              | VoiceOver-Sprache ein- oder ausschalten                 |
| ⌘⇧⇥ (Befehl-Umschalt-Tab)                             | Zur vorherigen App wechseln                            |
| ⌘⇥ (Befehl-Tab)                                       | Zur ursprünglichen App zurückwechseln                  |
| ←+→, dann Option + ← oder Option+→                    | Durch das Dock navigieren                              |
### Safari-Verknüpfungen

| Verknüpfung             | Aktion                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (Befehl-L)           | Ort öffnen                                       |
| ⌘T                       | Neuen Tab öffnen                                 |
| ⌘W                       | Aktuellen Tab schließen                          |
| ⌘R                       | Aktuellen Tab aktualisieren                      |
| ⌘.                       | Laden des aktuellen Tabs stoppen                 |
| ^⇥                       | Zum nächsten Tab wechseln                        |
| ^⇧⇥ (Strg-Umschalt-Tab) | Zum vorherigen Tab wechseln                      |
| ⌘L                       | Texteingabe/URL-Feld auswählen und bearbeiten     |
| ⌘⇧T (Befehl-Umschalt-T)  | Zuletzt geschlossenen Tab öffnen (mehrmals möglich) |
| ⌘\[                      | Eine Seite in der Browserverlauf zurückgehen     |
| ⌘]                       | Eine Seite in der Browserverlauf vorwärts gehen  |
| ⌘⇧R                      | Lesemodus aktivieren                             |

### Mail-Verknüpfungen

| Verknüpfung                   | Aktion                           |
| -------------------------- | ---------------------------- |
| ⌘L                         | Ort öffnen                    |
| ⌘T                         | Neuen Tab öffnen              |
| ⌘W                         | Aktuellen Tab schließen       |
| ⌘R                         | Aktuellen Tab aktualisieren   |
| ⌘.                         | Laden des aktuellen Tabs stoppen |
| ⌘⌥F (Befehl-Option/Alt-F) | In deinem Posteingang suchen |

# Referenzen

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)


<details>

<summary><strong>Lernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks bewerben möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegramm-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>
