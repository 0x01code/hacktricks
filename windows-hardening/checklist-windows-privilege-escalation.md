# Kontrolelys - Plaaslike Windows Privilege Escalation

<details>

<summary><strong>Leer AWS-hacking van nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

### **Beste gereedskap om te soek na Windows plaaslike privilege escalatie vektore:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

### [Stelsel Inligting](windows-local-privilege-escalation/#system-info)

* [ ] Verkry [**Stelsel inligting**](windows-local-privilege-escalation/#system-info)
* [ ] Soek na **kernel** [**uitbuitings deur skripte te gebruik**](windows-local-privilege-escalation/#version-exploits)
* [ ] Gebruik **Google om te soek** vir kernel **uitbuitings**
* [ ] Gebruik **searchsploit om te soek** vir kernel **uitbuitings**
* [ ] Interessante inligting in [**omgewingsveranderlikes**](windows-local-privilege-escalation/#environment)?
* [ ] Wagwoorde in [**PowerShell geskiedenis**](windows-local-privilege-escalation/#powershell-history)?
* [ ] Interessante inligting in [**Internet instellings**](windows-local-privilege-escalation/#internet-settings)?
* [ ] [**Hardeskywe**](windows-local-privilege-escalation/#drives)?
* [ ] [**WSUS-uitbuiting**](windows-local-privilege-escalation/#wsus)?
* [**AlwaysInstallElevated**](windows-local-privilege-escalation/#alwaysinstallelevated)?

### [Logging/AV enumerasie](windows-local-privilege-escalation/#enumeration)

* [ ] Kontroleer [**Oudit** ](windows-local-privilege-escalation/#audit-settings)en [**WEF** ](windows-local-privilege-escalation/#wef)instellings
* [ ] Kontroleer [**LAPS**](windows-local-privilege-escalation/#laps)
* [ ] Kontroleer of [**WDigest** ](windows-local-privilege-escalation/#wdigest)aktief is
* [ ] [**LSA-beskerming**](windows-local-privilege-escalation/#lsa-protection)?
* [ ] [**Credentials Guard**](windows-local-privilege-escalation/#credentials-guard)[?](windows-local-privilege-escalation/#cached-credentials)
* [ ] [**Gekasde Wagwoorde**](windows-local-privilege-escalation/#cached-credentials)?
* [ ] Kontroleer of enige [**AV**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/windows-av-bypass/README.md)
* [ ] [**AppLocker-beleid**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs/README.md#applocker-policy)?
* [**UAC**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs/uac-user-account-control/README.md)
* [**Gebruikerbevoegdhede**](windows-local-privilege-escalation/#users-and-groups)
* Kontroleer [**huidige** gebruiker **bevoegdhede**](windows-local-privilege-escalation/#users-and-groups)
* Is jy 'n [**lid van enige bevoorregte groep**](windows-local-privilege-escalation/#privileged-groups)?
* Kontroleer of jy enige van hierdie tokens geaktiveer het: **SeImpersonatePrivilege, SeAssignPrimaryPrivilege, SeTcbPrivilege, SeBackupPrivilege, SeRestorePrivilege, SeCreateTokenPrivilege, SeLoadDriverPrivilege, SeTakeOwnershipPrivilege, SeDebugPrivilege** ?
* [**Gebruikersessies**](windows-local-privilege-escalation/#logged-users-sessions)?
* Kontroleer[ **gebruikershuise**](windows-local-privilege-escalation/#home-folders) (toegang?)
* Kontroleer [**Wagwoordbeleid**](windows-local-privilege-escalation/#password-policy)
* Wat is[ **binne die Knipbord**](windows-local-privilege-escalation/#get-the-content-of-the-clipboard)?

### [Netwerk](windows-local-privilege-escalation/#network)

* Kontroleer **huidige** [**netwerk** **inligting**](windows-local-privilege-escalation/#network)
* Kontroleer **versteekte plaaslike dienste** wat beperk is tot die buitewêreld

### [Lopende Prosesse](windows-local-privilege-escalation/#running-processes)

* Prosesse binêre [**lêer en vouer toestemmings**](windows-local-privilege-escalation/#file-and-folder-permissions)
* [**Geheue Wagwoordontginning**](windows-local-privilege-escalation/#memory-password-mining)
* [**Onveilige GUI-toepassings**](windows-local-privilege-escalation/#insecure-gui-apps)
* Steel geloofsbriewe met **interessante prosesse** via `ProcDump.exe` ? (firefox, chrome, ens ...)

### [Dienste](windows-local-privilege-escalation/#services)

* [Kan jy enige diens **verander**?](windows-local-privilege-escalation/#permissions)
* [Kan jy die **binêre** wat deur enige **diens** **uitgevoer** word, **verander**?](windows-local-privilege-escalation/#modify-service-binary-path)
* [Kan jy die **register** van enige **diens** **verander**?](windows-local-privilege-escalation/#services-registry-modify-permissions)
* [Kan jy voordeel trek uit enige **ongekwoteerde diens** binêre **pad**?](windows-local-privilege-escalation/#unquoted-service-paths)
### [**Aansoeke**](windows-local-privilege-escalation/#applications)

* [ ] **Skryf** [**regte op geïnstalleerde aansoeke**](windows-local-privilege-escalation/#write-permissions)
* [ ] [**Begin Aansoeke**](windows-local-privilege-escalation/#run-at-startup)
* [ ] **Kwesbare** [**Drywers**](windows-local-privilege-escalation/#drivers)

### [DLL Ontvoering](windows-local-privilege-escalation/#path-dll-hijacking)

* [ ] Kan jy **skryf in enige vouer binne die PAD**?
* [ ] Is daar enige bekende diens binêre wat **probeer om enige nie-bestaande DLL te laai**?
* [ ] Kan jy **skryf** in enige **binêre vouer**?

### [Netwerk](windows-local-privilege-escalation/#network)

* [ ] Enumereer die netwerk (aandele, koppelvlakke, roetes, bure, ...)
* [ ] Neem 'n spesiale kyk na netwerkdienste wat luister op die plaaslike gasheer (127.0.0.1)

### [Windows Geloofsbriewe](windows-local-privilege-escalation/#windows-credentials)

* [ ] [**Winlogon** ](windows-local-privilege-escalation/#winlogon-credentials)geloofsbriewe
* [ ] [**Windows Vault**](windows-local-privilege-escalation/#credentials-manager-windows-vault) geloofsbriewe wat jy kan gebruik?
* [ ] Interessante [**DPAPI geloofsbriewe**](windows-local-privilege-escalation/#dpapi)?
* [ ] Wagwoorde van gestoorde [**Wifi-netwerke**](windows-local-privilege-escalation/#wifi)?
* [ ] Interessante inligting in [**gestoorde RDP-koppeling**](windows-local-privilege-escalation/#saved-rdp-connections)?
* [ ] Wagwoorde in [**onlangs uitgevoerde bevele**](windows-local-privilege-escalation/#recently-run-commands)?
* [ ] [**Remote Desktop Credentials Manager**](windows-local-privilege-escalation/#remote-desktop-credential-manager) wagwoorde?
* [ ] [**AppCmd.exe** bestaan](windows-local-privilege-escalation/#appcmd-exe)? Geloofsbriewe?
* [ ] [**SCClient.exe**](windows-local-privilege-escalation/#scclient-sccm)? DLL Sybelading?

### [Lêers en Registreer (Geloofsbriewe)](windows-local-privilege-escalation/#files-and-registry-credentials)

* [ ] **Putty:** [**Geloofsbriewe**](windows-local-privilege-escalation/#putty-creds) **en** [**SSH-gashere sleutels**](windows-local-privilege-escalation/#putty-ssh-host-keys)
* [ ] [**SSH-sleutels in die register**](windows-local-privilege-escalation/#ssh-keys-in-registry)?
* [ ] Wagwoorde in [**ongeagte lêers**](windows-local-privilege-escalation/#unattended-files)?
* [ ] Enige [**SAM & SYSTEM**](windows-local-privilege-escalation/#sam-and-system-backups) rugsteun?
* [ ] [**Wolk geloofsbriewe**](windows-local-privilege-escalation/#cloud-credentials)?
* [ ] [**McAfee SiteList.xml**](windows-local-privilege-escalation/#mcafee-sitelist.xml) lêer?
* [ ] [**Gekas GPP Wagwoord**](windows-local-privilege-escalation/#cached-gpp-pasword)?
* [ ] Wagwoord in [**IIS Web konfigurasie lêer**](windows-local-privilege-escalation/#iis-web-config)?
* [ ] Interessante inligting in [**web** **log**](windows-local-privilege-escalation/#logs)?
* [ ] Wil jy [**vra vir geloofsbriewe**](windows-local-privilege-escalation/#ask-for-credentials) van die gebruiker?
* [ ] Interessante [**lêers binne die Stortbak**](windows-local-privilege-escalation/#credentials-in-the-recyclebin)?
* [ ] Ander [**register wat geloofsbriewe bevat**](windows-local-privilege-escalation/#inside-the-registry)?
* [ ] Binne [**Blaaierdata**](windows-local-privilege-escalation/#browsers-history) (databasisse, geskiedenis, bladmerke, ...)?
* [ ] [**Generiese wagwoordsoektog**](windows-local-privilege-escalation/#generic-password-search-in-files-and-registry) in lêers en register
* [ ] [**Hulpmiddels**](windows-local-privilege-escalation/#tools-that-search-for-passwords) om outomaties te soek na wagwoorde

### [Uitgelekte Handlers](windows-local-privilege-escalation/#leaked-handlers)

* [ ] Het jy toegang tot enige handler van 'n proses wat deur 'n administrateur uitgevoer word?

### [Pyp Kliënt Impersonasie](windows-local-privilege-escalation/#named-pipe-client-impersonation)

* [ ] Kontroleer of jy dit kan misbruik

**Probeer Hard Sekuriteitsgroep**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Leer AWS hak van nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien jou **maatskappy geadverteer in HackTricks** of **laai HackTricks af in PDF** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PRs in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
