# Anti-Forensiese Tegnieke

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy jou **maatskappy geadverteer wil sien in HackTricks** of **HackTricks in PDF wil aflaai** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling van eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>

## Tydstempels

'n Aanvaller kan belangstel om **die tydstempels van lêers te verander** om opsporing te vermy.\
Dit is moontlik om die tydstempels binne die MFT in eienskappe `$STANDARD_INFORMATION` en `$FILE_NAME` te vind.

Beide eienskappe het 4 tydstempels: **Wysiging**, **toegang**, **skepping**, en **MFT-registerwysiging** (MACE of MACB).

**Windows verkenner** en ander gereedskap toon die inligting vanaf **`$STANDARD_INFORMATION`**.

### TimeStomp - Anti-forensiese Gereedskap

Hierdie gereedskap **verander** die tydstempelinligting binne **`$STANDARD_INFORMATION`** **maar nie** die inligting binne **`$FILE_NAME`** nie. Daarom is dit moontlik om **verdagte aktiwiteit te identifiseer**.

### Usnjrnl

Die **USN Joernaal** (Update Sequence Number Journal) is 'n kenmerk van die NTFS (Windows NT-lêersisteem) wat volume-veranderings byhou. Die [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) gereedskap maak dit moontlik om hierdie veranderings te ondersoek.

![](<../../.gitbook/assets/image (801).png>)

Die vorige beeld is die **uitset** wat deur die **gereedskap** getoon word waar dit waargeneem kan word dat sekere **veranderings aan die lêer gedoen is**.

### $LogFile

**Alle metadata-veranderings aan 'n lêersisteem word gelog** in 'n proses wat bekend staan as [write-ahead logging](https://en.wikipedia.org/wiki/Write-ahead\_logging). Die gelogde metadata word in 'n lêer genaamd `**$LogFile**` gehou, wat in die hoofgids van 'n NTFS-lêersisteem geleë is. Gereedskap soos [LogFileParser](https://github.com/jschicht/LogFileParser) kan gebruik word om hierdie lêer te ontled en veranderings te identifiseer.

![](<../../.gitbook/assets/image (137).png>)

Weereens, in die uitset van die gereedskap is dit moontlik om te sien dat **sekere veranderings uitgevoer is**.

Met dieselfde gereedskap is dit moontlik om te identifiseer **watter tyd die tydstempels verander is**:

![](<../../.gitbook/assets/image (1089).png>)

* CTIME: Lêer se skeppingstyd
* ATIME: Lêer se wysigingstyd
* MTIME: Lêer se MFT-registerwysiging
* RTIME: Lêer se toegangstyd

### `$STANDARD_INFORMATION` en `$FILE_NAME` vergelyking

'n Ander manier om verdagte veranderde lêers te identifiseer sou wees om die tyd op beide eienskappe te vergelyk en te soek na **verskille**.

### Nanosekondes

**NTFS**-tydstempels het 'n **presisie** van **100 nanosekondes**. Daarom is dit baie verdag as lêers met tydstempels soos 2010-10-10 10:10:**00.000:0000 gevind word**.

### SetMace - Anti-forensiese Gereedskap

Hierdie gereedskap kan beide eienskappe `$STARNDAR_INFORMATION` en `$FILE_NAME` verander. Tog, vanaf Windows Vista, is dit nodig vir 'n lewende OS om hierdie inligting te verander.

## Data Versteek

NTFS gebruik 'n groep en die minimum inligtingsgrootte. Dit beteken dat as 'n lêer 'n groep en 'n half gebruik, sal die **oorskietende helfte nooit gebruik word nie** totdat die lêer uitgevee word. Daarom is dit moontlik om data in hierdie oorskietende spasie te **versteek**.

Daar is gereedskap soos slacker wat dit moontlik maak om data in hierdie "versteekte" spasie te versteek. Nietemin, 'n ontleding van die `$logfile` en `$usnjrnl` kan wys dat daar data bygevoeg is:

![](<../../.gitbook/assets/image (1060).png>)

Dit is dus moontlik om die oorskietende spasie te herwin deur gereedskap soos FTK Imager te gebruik. Let daarop dat hierdie soort gereedskap die inhoud geobskureer of selfs versleutel kan stoor.

## UsbKill

Dit is 'n gereedskap wat die rekenaar sal **afsluit as enige verandering in die USB**-poorte opgespoor word.\
'n Manier om dit te ontdek sou wees om die lopende prosesse te ondersoek en **elke Python-skrip wat loop te hersien**.

## Lewende Linux Verspreidings

Hierdie verspreidings word **uitgevoer binne die RAM**-geheue. Die enigste manier om hulle op te spoor is **indien die NTFS-lêersisteem met skryfregte aangeheg is**. As dit net met leesregte aangeheg is, sal dit nie moontlik wees om die indringing op te spoor nie.

## Veilige Skrapping

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

## Windows Konfigurasie

Dit is moontlik om verskeie Windows-loggingsmetodes uit te skakel om die forensiese ondersoek baie moeiliker te maak.

### Skakel Tydstempels Af - UserAssist

Dit is 'n register sleutel wat datums en ure behou wanneer elke uitvoerbare program deur die gebruiker uitgevoer is.

Die afskakeling van UserAssist vereis twee stappe:

1. Stel twee register sleutels, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` en `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, albei na nul om aan te dui dat ons wil hê UserAssist afgeskakel moet word.
2. Maak jou register-subbome skoon wat lyk soos `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`.

### Skakel Tydstempels Af - Prefetch

Dit sal inligting oor die uitgevoerde toepassings stoor met die doel om die Windows-stelsel se prestasie te verbeter. Dit kan egter ook nuttig wees vir forensiese praktyke.

* Voer `regedit` uit
* Kies die lêerpad `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Regskliek op beide `EnablePrefetcher` en `EnableSuperfetch`
* Kies Wysig op elkeen van hierdie om die waarde van 1 (of 3) na 0 te verander
* Herlaai

### Skakel Tydstempels Af - Laaste Toegangstyd

Telkens wanneer 'n gids vanaf 'n NTFS-volume op 'n Windows NT-bediener geopen word, neem die stelsel die tyd om **'n tydstempelveld op elke gelysde gids by te werk**, genaamd die laaste toegangstyd. Op 'n baie gebruikte NTFS-volume kan dit die prestasie beïnvloed.

1. Maak die Registerredakteur oop (Regedit.exe).
2. Blaai na `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Soek na `NtfsDisableLastAccessUpdate`. As dit nie bestaan nie, voeg hierdie DWORD by en stel sy waarde op 1 om die proses te deaktiveer.
4. Maak die Registerredakteur toe en herlaai die bediener.
### Verwyder USB Geskiedenis

Al die **USB-toestelinskrywings** word gestoor in die Windows-register onder die **USBSTOR** register sleutel wat sub sleutels bevat wat geskep word wanneer jy 'n USB-toestel in jou rekenaar of draagbare rekenaar insteek. Jy kan hierdie sleutel vind by `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Deur hierdie** te verwyder, sal jy die USB-geskiedenis verwyder.\
Jy kan ook die hulpmiddel [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) gebruik om seker te maak dat jy hulle verwyder het (en om hulle te verwyder).

'n Ander lêer wat inligting oor die USB's stoor is die lêer `setupapi.dev.log` binne `C:\Windows\INF`. Dit moet ook verwyder word.

### Deaktiveer Skadukopieë

**Lys** skadukopieë met `vssadmin list shadowstorage`\
**Verwyder** hulle deur `vssadmin delete shadow` uit te voer

Jy kan hulle ook via die GUI verwyder deur die stappe te volg wat voorgestel word by [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Om skadukopieë te deaktiveer [stappe vanaf hier](https://support.waters.com/KB\_Inf/Other/WKB15560\_How\_to\_disable\_Volume\_Shadow\_Copy\_Service\_VSS\_in\_Windows):

1. Maak die Dienste-program oop deur "dienste" in die teks-soekblok in te tik nadat jy op die Windows begin-knoppie geklik het.
2. Vind "Volume Shadow Copy" in die lys, kies dit, en kry toegang tot Eienskappe deur regs te klik.
3. Kies "Gedeaktiveer" vanaf die "Beginsoort" keuselyste, en bevestig dan die verandering deur op Toepas en OK te klik.

Dit is ook moontlik om die konfigurasie te wysig van watter lêers in die skadukopie gekopieer gaan word in die register `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

### Oorskryf verwyderde lêers

* Jy kan 'n **Windows-hulpmiddel** gebruik: `cipher /w:C` Dit sal cipher aandui om enige data van die beskikbare ongebruikte skyfspasie binne die C-aandrywing te verwyder.
* Jy kan ook hulpmiddels soos [**Eraser**](https://eraser.heidi.ie) gebruik

### Verwyder Windows gebeurtenislogs

* Windows + R --> eventvwr.msc --> Brei "Windows Logs" uit --> Regs klik op elke kategorie en kies "Log Skoonmaak"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

### Deaktiveer Windows gebeurtenislogs

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* Binne die dienste-afdeling, deaktiveer die diens "Windows Event Log"
* `WEvtUtil.exec clear-log` of `WEvtUtil.exe cl`

### Deaktiveer $UsnJrnl

* `fsutil usn deletejournal /d c:`

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kontroleer die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS-familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFT's**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>
